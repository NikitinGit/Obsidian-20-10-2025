# WebSocket / STOMP в StrikerStat

Уведомления судьям/организаторам (старт боя, результат, добавлен/удалён бой следующего раунда и т.д.) идут по WebSocket поверх протокола STOMP. Ниже — разбор фронта и бэка в формате вопрос/ответ.

---

## Архитектура

>[!question]- как вообще устроены уведомления: фронт → бэкенд
> **Транспорт:** WebSocket, поверх него протокол **STOMP** (текстовый: CONNECT/SUBSCRIBE/SEND/MESSAGE/heartbeat).
> 
> **Фронт:** библиотека `webstomp-client` (`frontend/services/webSocketService.js`). Подключается к `/ws`, подписывается на персональный канал `/topic/{userId}`.
> 
> **Бэкенд (Spring):** `@EnableWebSocketMessageBroker`, простой in-memory брокер `enableSimpleBroker("/topic")`. Конфиг — `config/websocket/WebSocketConfig.java`.
> 
> **Путь на проде:** браузер → `wss://домен:443/ws` → **nginx** (TLS, проксирование) → (возможно **edge Timeweb**) → бэкенд `127.0.0.1:6300`.
> **Путь локально:** браузер → `ws://localhost:6300/ws` **напрямую в бэкенд** (без nginx) — поэтому локально проблем с обрывами нет.

>[!question]- как бэкенд отправляет уведомление конкретному судье
> Через `SimpMessagingTemplate.convertAndSend("/topic/" + userId, message)`.
> Центральный сервис — `service/judging/JudgeNotificationService`. Все публичные методы (`sendBattleNotificationToMainJudges`, `...Except`, `sendBattleCreatedNotification` и т.д.) сводятся к приватному `sendNotification(eventId, judgeUserIds, message)`, который:
> 1. шлёт сообщение каждому судье в `/topic/{id}`;
> 2. инвалидирует event-кэши через `EventCacheEvictor` (чтобы фронт по уведомлению перечитал свежие данные).
> 
> Получатели вычисляются live-запросом в БД (`EventBidJudgeRepository.getJudgeUserIds/getAllMainJudgeUserIds`) — **не кэшируются**.

>[!question]- зачем вообще nginx для WebSocket
> nginx — reverse-proxy для **всего** приложения, WS идёт через него заодно. Нужен для:
> 1. **TLS-терминации** — бэкенд слушает голый `127.0.0.1:6300` без HTTPS; браузер ходит по `wss://`. nginx расшифровывает.
> 2. **Единый порт/домен 443** — `/` → фронт (Nuxt :3000), `/ws` → бэкенд :6300, `/api` и т.д. Один origin → нет CORS, не нужно открывать порты наружу.
> 3. **Безопасность** — бэкенд на `127.0.0.1`, недоступен из интернета напрямую.
> 
> WS — особый случай: это долгоживущее соединение, начинающееся как HTTP-запрос с `Upgrade: websocket`. nginx надо ЯВНО научить его держать:
> ```nginx
> proxy_http_version 1.1;
> proxy_set_header Upgrade $http_upgrade;
> proxy_set_header Connection 'upgrade';
> proxy_read_timeout 3600s;   # иначе закроет "тихое" долгое соединение
> ```
> Можно было бы пустить WS мимо nginx (отдельный порт+TLS на бэке), но это сломало бы единый origin и потребовало сертификаты на бэке — так не делают.

---

## Heartbeat и логи

>[!question]- что такое PING/PONG в консоли и откуда они берутся
> Это **heartbeat** STOMP 1.1+ — служебные «пульсы» (1-байтный `\n`), которыми клиент и сервер подтверждают, что соединение живо. Согласуются при CONNECT: `heart-beat:10000,10000` = слать каждые 10с и ждать каждые 10с.
> 
> Логи печатает сам модуль **`webstomp-client`** (в `node_modules/webstomp-client/`) через свой `debug()` = обычный `console.log`:
> - `>>> PING` — клиент отправил пульс (`_setupHeartbeat` → `setInterval`);
> - `<<< PONG` — пришёл одиночный `\n` от сервера (активность сервера);
> - `>>> CONNECT/SUBSCRIBE`, `<<< MESSAGE`, `Whoops! Lost connection...`.
> 
> Логи идут **в консоль браузера**, никаких файлов в node_modules не создаётся. Чтобы убрать спам — создавать клиент с `webstomp.over(socket, { debug: false })`.

>[!question]- что за модуль webstomp-client
> STOMP-клиент поверх WebSocket для браузера и Node.js, чистый JS. В `package.json`: `"webstomp-client": "^1.2.6"`.
> - Репозиторий/README: https://github.com/JSteunou/webstomp-client
> - npm: https://www.npmjs.com/package/webstomp-client
> - Локально: `node_modules/webstomp-client/src/client.js` (вся логика, ~370 строк), `index.d.ts` (типы = карта API).
> 
> Модуль старый и почти не развивается. Современный аналог с авто-reconnect из коробки — **`@stomp/stompjs`** (https://github.com/stomp-js/stompjs); если переписывать WS-клиент — смотреть в его сторону (тогда наш ручной reconnect стал бы не нужен).

>[!question]- конфиг heartbeat на бэкенде (Spring)
> `config/websocket/WebSocketConfig.java`:
> ```java
> config.enableSimpleBroker("/topic")
>       .setHeartbeatValue(new long[]{10000, 10000})
>       .setTaskScheduler(brokerTaskScheduler());   // ОБЯЗАТЕЛЬНО, иначе heartbeat сервера не работает
> ```
> Endpoint `/ws` с `WebSocketHandshakeInterceptor`; `/ws-public` без авторизации.
> Без `TaskScheduler` сервер не шлёт/не ждёт heartbeat — соединение молча умирает. Здесь он задан, так что конфиг корректный.

---

## Ошибки и обрывы

>[!question]- что значит "WebSocket is already in CLOSING or CLOSED state"
> Это **клиентская** ошибка браузера: JS вызвал `ws.send()` на сокете, который уже закрыт/закрывается.
> Это НЕ сам обрыв, а **следствие**: соединение уже умерло, а код продолжает в него писать.
> Кто писал в мёртвый сокет:
> - раньше (баг) — протёкшие heartbeat-`setInterval` от «зомби»-клиентов (см. ниже);
> - после фикса — наш `teardownClient()` слал `DISCONNECT` в уже закрытый сокет (тоже поправили — слать только если сокет OPEN).
> Сама ошибка НЕ говорит, кто оборвал соединение.

>[!question]- что значит code 1006 / wasClean:false
> `1006` = **аварийный обрыв TCP без close-фрейма**. Если бы закрывал штатно клиент или сервер — был бы чистый код (1000/1008) и close-фрейм.
> `1006` означает, что соединение физически разорвали в середине — почерк **прослойки (прокси/балансировщик/сеть)** или резкого дропа.

>[!question]- что значит "Invalid frame header"
> Браузерная ошибка уровня WS-протокола: пришедшие байты **не сложились в валидный WebSocket-кадр** (нарушен формат по RFC 6455).
> Значит в WS-туннель вклинилось что-то не-WebSocket или кадры покорёжены. Причины:
> - **прослойка (edge/прокси) портит поток** — вставляет HTTP-чанк, режет кадр, неправильно буферизует (усиливает гипотезу про edge);
> - рассогласование `permessage-deflate` (WS-сжатие);
> - кривое проксирование апгрейда (HTTP/2 vs WS).
> Не уязвимость — функциональный сбой соединения (WS не встаёт / рвётся → reconnect).

>[!question]- может ли потеряться WS-сообщение при разрыве соединения
> **ДА.** Spring `SimpleBroker` доставляет сообщение только сессиям, подключённым ПРЯМО СЕЙЧАС. Он **не хранит и не переотправляет** офлайн-клиенту.
> Любое уведомление (`convertAndSend`), отправленное в момент, когда сокет в разрыве/reconnect (включая ~5с задержки переподключения), **теряется безвозвратно**. Нет ни очередей, ни replay, ни redelivery по ACK.
> При нестабильном соединении окна потери возникают постоянно → это и есть «уведомления не приходят».
> 
> **Решения:**
> 1. **Дозапрос состояния после reconnect (самое дешёвое и надёжное):** при успешном переподключении фронт сам перечитывает данные (сетку/бои), а не ждёт повторной доставки. Тогда потеря уведомления некритична.
> 2. Чинить стабильность WS (edge/nginx) — снижает частоту, но не исключает.
> 3. Брокер с очередями (RabbitMQ STOMP) вместо SimpleBroker — гарантированная доставка, но большая переделка.

---

## Фронтовый баг с reconnect (исправлен)

>[!question]- почему в консоли была лавина "already CLOSING or CLOSED" и растущий "did not receive server activity"
> Баг в `frontend/services/webSocketService.js`: при reconnect создавался новый webstomp-клиент, но старый **не уничтожался**. Причина — мы **затирали `ws.onclose` библиотеки** своим обработчиком:
> ```js
> stompClient.value.ws.onclose = (event) => { tryReconnect(account); };  // ПЛОХО
> ```
> Внутренний `onclose` webstomp вызывает `_cleanUp()`, который гасит heartbeat-интервалы (`pinger`/`ponger`). Раз мы его перезатёрли — очистка не происходила, старые `setInterval` жили вечно и слали PING в мёртвые сокеты. Каждый обрыв добавлял ещё один «зомби» → лавина ошибок, а счётчик `did not receive server activity` рос у протухшего инстанса.

>[!question]- как починили reconnect на фронте
> В `webSocketService.js`:
> 1. **Не затираем `onclose` webstomp, а оборачиваем** — сперва вызываем родной (его `_cleanUp` гасит heartbeat), потом reconnect:
>    ```js
>    const webstompOnClose = socket.onclose;
>    socket.onclose = (event) => {
>      if (typeof webstompOnClose === "function") webstompOnClose.call(socket, event);
>      if (stompClient.value === client) { tryReconnect(account); }  // только для актуального клиента
>    };
>    ```
> 2. **`teardownClient()`** перед созданием нового соединения: снимает обработчики со старого сокета, `disconnect()` только если сокет OPEN (иначе webstomp уже сделал `_cleanUp` в onclose), закрывает.
> 3. `disconnectWebsocket()` чистит и `reconnectTimeout`, и клиента безусловно.
> 
> Результат: один клиент = один heartbeat-таймер, reconnect чистый, спам и зомби ушли.

---

## Авторизация WS

>[!question]- почему прямой websocat/wscat к бэкенду даёт 200 вместо 101
> На `/ws` висит `WebSocketHandshakeInterceptor` (`config/websocket/`). В `beforeHandshake` он требует cookie-токен `strikerstat_token`; без валидной куки возвращает `false` → Spring НЕ делает upgrade → отдаёт `200`. В коде даже комментарий: «невозможно подключиться через терминал: wscat -c ws://localhost:6300/ws».
> Чтобы протестировать напрямую — передать cookie из браузера (DevTools → Application → Cookies):
> ```bash
> websocat -v --header='Cookie: strikerstat_token=ЗНАЧЕНИЕ' ws://127.0.0.1:6300/ws
> ```
> Успех = в ответе `Upgrade: websocket` + `Sec-WebSocket-Accept` (а не 200).

---

## Связь с кэшом

>[!question]- как WS-уведомления связаны с кэшом и почему фронт показывал старые данные
> Олимпийская сетка и таблицы кэшируются (`OLYMPIC_ELIMINATION_BRACKETS`, `OLYMPIC_ALL_TABLES`, `OLYMPIC_WINNER_TABLES`, `OPEN_EVENT_DTO`, все по ключу `#eventId`; сетка — для всех, кроме организатора).
> Раньше при проставлении результата кэш сетки **не сбрасывался** → судья получал WS-уведомление «обновите страницу», перечитывал сетку и видел СТАРЫЕ данные из кэша.
> 
> **Фикс:** `config/EventCacheEvictor` — программная инвалидация event-кэшей по `eventId`, вызывается из точек отправки WS-уведомлений (`JudgeNotificationService.sendNotification`, и в `VideoOverlayService` для старт/завершение/отмена боя). Эвикт делается **после коммита транзакции** (`TransactionSynchronization.afterCommit`), иначе конкурентное чтение успело бы вернуть в кэш старые данные до коммита.
> Декларативный `@CacheEvict` тут не годится — методы-отправители вызываются из множества мест и в т.ч. self-invocation, поэтому через `CacheManager` в коде.

---

## Диагностика обрывов на проде

>[!question]- как локализовать, КТО рвёт WebSocket: бэкенд / nginx / edge Timeweb
> Подробная методика (тесты по слоям, websocat/python с heartbeat, чтение логов nginx) — в памятке **NGINX.md** (блок «как локализовать, КТО рвёт WebSocket»).
> Кратко: подключаться к WS на разной «глубине» и засекать LIFETIME:
> 1. `ws://127.0.0.1:6300/ws` — прямо в бэкенд (мимо nginx+edge);
> 2. `wss://127.0.0.1:443/ws` — через локальный nginx (мимо edge);
> 3. `wss://домен/ws` — весь путь, как браузер.
> Где начинает рваться — там виновник.

>[!question]- ИТОГ диагностики StrikerStat (что выяснили про обрывы на препроде)
> **Симптом:** на препроде WS рвётся стабильно ~20-30с (`1006`, потом и `Invalid frame header`), по кругу. Локально проблемы нет.
> 
> **Три факта сошлись (бэкенд исправен, рубит прослойка):**
> 
> | Сценарий | Путь | Результат |
> |---|---|---|
> | Локально, браузер | прямо в `:6300` | WS живёт долго ✅ |
> | Препрод, `websocat` | прямо в `127.0.0.1:6300` | держался 5+ мин ✅ |
> | Препрод, браузер | через nginx + edge | рвётся ~20с ❌ |
> 
> **Вывод:** виноват слой между браузером и бэкендом, который есть ТОЛЬКО на препроде. nginx-конфиг `/ws` корректный (`Upgrade`/`Connection`/`http_version 1.1`), но без `proxy_read_timeout` (дефолт 60с), а рвётся на 20с → ставка на **edge Timeweb**. `Invalid frame header` (порча кадров) дополнительно указывает на вмешательство прослойки.
> 
> **План:**
> 1. Добавить в nginx `location /ws`: `proxy_read_timeout 3600s; proxy_send_timeout 3600s;` → снять nginx с подозрений.
> 2. Если рвёт всё равно → edge Timeweb (панель/поддержка про лимит времени жизни WS).
> 3. Проверить наличие edge: `dig +short домен` vs IP VM (`ip -4 addr`); лишние заголовки в `curl -sI`; видит ли nginx реальный client-IP в access.log.
> 4. На фронте — авто-дозапрос состояния после reconnect (делает потерю уведомлений некритичной независимо от того, починили ли обрыв).