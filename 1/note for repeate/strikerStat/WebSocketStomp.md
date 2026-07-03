
1. [ ] при подписке фронта на топик что происходит - 1) запрос на подключение к серверу по http 1.1 , nginx меняет протокол http и перенаправляет на подключение на бэк java, бэк java отправляет обратно на прямик на фронт ответ на подключение и после удачного подключения  коннект держится метежду фронтом и бэком без участия nginx и пакеты от фронта на бэк java  и обратно отправляются на прямик минуя nginx ? - нет , бэк общается только с nginx , коннект  начинается с 
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

---

## Тестирование разрыва соединения

>[!question]- как локально симулировать разрыв WS (что НЕ работает)
> - **DevTools → Network → Offline** — у Chromium/Brave НЕ роняет уже открытый WebSocket-туннель надёжно, для теста heartbeat-обрыва не годится.
> - `navigator.onLine` — про физический сетевой интерфейс, а не про состояние вкладки/таймеров. На реальные обрывы WS не завязываться.
> 
> **Что работает:**
> - **Точка останова в дебаге Spring** (Suspend: All) — замораживает в т.ч. брокерный поток heartbeat (см. ниже).
> - **Дроп TCP на уровне ОС:** `iptables -A INPUT -p tcp --dport 6300 -j DROP` (включить), `iptables -D INPUT ... -j DROP` (выключить — тот же spec, отличается только `-A`→`-D`). Требует рабочего `sudo`/root.

>[!question]- что происходит, если поставить breakpoint (Suspend: All) перед отправкой уведомления
> Это замораживает ВЕСЬ сервер, включая поток брокера, который шлёт heartbeat. Сервер перестаёт слать свои `\n` каждые 10с. Дальше на фронте срабатывает **клиентский** защитный механизм:
> 
> У `webstomp-client` два независимых `setInterval`:
> 1. **pinger** — каждые 10с шлёт `>>> PING` (`>>> length 1` = кадр heartbeat в 1 байт `\n`). «Я живой».
> 2. **ponger** — каждые 10с проверяет `delta = Date.now() - serverActivity`. Если `delta > _ttl*2` (~20с молчания сервера) → логирует `did not receive server activity for the last Nms` и **сам вызывает `ws.close()`**.
> 
> Поэтому **через ~20-30с (2 пропущенных серверных heartbeat) соединение закрывает САМ КЛИЕНТ**, а не сеть и не сервер. Это штатное поведение heartbeat: «сервер молчит дольше двойного интервала → считаю соединение мёртвым».
> 
> **ВАЖНО:** этот тест воспроизводит «сервер перестал слать heartbeat», а НЕ «у клиента пропал интернет». В реальной жизни брейкпоинт в бизнес-логике (не Suspend: All) не заморозил бы брокерный поток, и клиент бы не отвалился.

>[!question]- почему после ws.close() идёт лавина "WebSocket is already in CLOSING or CLOSED state"
> `ws.close()` (вызванный ponger'ом) перевёл сокет в CLOSING/CLOSED, **но pinger-таймер ещё не остановлен** — `_cleanUp()` (чистит оба интервала) вызывается из `ws.onclose`, который срабатывает не мгновенно. Поэтому pinger по таймеру продолжает дёргать `_wsSend(PING)` в мёртвый сокет → каждый раз `WebSocket is already in CLOSING or CLOSED state`.
> Числа в `did not receive server activity` растут шагом ~10000мс (24613 → 34613 → 44613) — это тики таймеров раз в 10с.

>[!question]- может ли фоновая вкладка браузера сама уронить WS (переключился на другую вкладку/окно на 30с)
> **ДА, это частый бытовой кейс.** Heartbeat webstomp построен на `setInterval`, а Chromium/Brave **агрессивно тормозят таймеры скрытой вкладки**:
> 1. Вкладка скрыта → таймеры ограничены ~1 раз/сек.
> 2. Скрыта > 5 мин → "intensive throttling", таймеры будятся ~1 раз/мин.
> 3. **Freezing** (нехватка ресурсов / energy saver) → вкладка заморожена, JS не исполняется вообще.
> 
> Сам WebSocket — это TCP на уровне ОС, он не зависит от исполнения JS. Но heartbeat — чистый JS на таймерах. В фоне:
> - **pinger притормозили** → фронт перестаёт слать `\n` → **Spring через ~20с молчания клиента сам закроет соединение** как мёртвое (основной механизм обрыва в этом кейсе).
> - **ponger притормозили** → клиент узнаёт о смерти сокета с задержкой.
> 
> Когда вернёшься во вкладку — она «размёрзнется», webstomp получит `onclose` → сработает наш `tryReconnect`.
> 
> **Следствие:** за время фоновой вкладки уведомления, отправленные сервером, **теряются безвозвратно** (SimpleBroker не буферизирует). Это усиливает аргумент за дозапрос состояния после reconnect.

>[!question]- как правильно чинить фоновый сон вкладки
> - НЕ увеличивать heartbeat-интервалы — замороженная вкладка не тикает вообще, любой интервал бессмыслен.
> - НЕ полагаться на `navigator.onLine`.
> - **Page Visibility API:** ловить `visibilitychange`; при возврате вкладки (`document.visibilityState === 'visible'`) принудительно проверять `stompClient`/`readyState`, при необходимости форсить reconnect + дозапрос `eventBus.getAllDataFromServer()`. Надёжнее, чем ждать пробуждения heartbeat-таймеров.
> 
> Это закрывает кейс «переключился на другую вкладку на 30с» независимо от причины обрыва.

---

## Гарантия доставки (вернуться позже)

>[!question]- гарантирует ли SimpleBroker доставку при разрыве соединения
> **НЕТ.** `SimpleBroker` (`enableSimpleBroker`) — in-memory брокер **без персистентности**. У него нет:
> - **очередей** — не складывает сообщение «на потом»;
> - **подтверждений (ACK)** — не ждёт, что клиент получил;
> - **переотправки (redelivery)** — не повторяет недоставленное;
> - **хранилища** — всё в памяти процесса, рестарт бэка = всё потеряно.
> 
> Алгоритм `convertAndSend("/topic/{id}", msg)` буквально: «найти среди **сейчас подключённых** сессий подписанные на `/topic/{id}` и записать им в сокет». Сессия в разрыве/reconnect → сообщение **некому отдать, оно исчезает**. Это by design: `/topic` — pub/sub «здесь и сейчас», как радиоэфир.

>[!question]- как добавить гарантию доставки (4 варианта по возрастанию сложности)
> **1. Дозапрос состояния после reconnect (рекомендую — самое дешёвое).**
> Не гарантировать доставку *сообщения*, а сделать потерю некритичной. Уведомления StrikerStat — сигнал «данные изменились, перечитай»; состояние живёт в БД и доступно по REST. При каждом успешном reconnect (и возврате фоновой вкладки) фронт сам дёргает `eventBus.getAllDataFromServer()`. Тогда неважно, сколько уведомлений потерялось — после переподключения картина актуальна.
> *+* ~10 строк на фронте, ноль изменений на бэке, надёжно против любой причины обрыва.
> *−* это «гарантия актуальности состояния», а не «гарантия доставки сообщения». Для refresh-сигналов достаточно.
> 
> **2. Версия состояния + поллинг (уже частично есть).**
> В `battleService.js` есть `getEventVersion` — поллинг `event/version` раз в 5с, сравнение версии. Запасной канал, не зависящий от WS: версия выросла → «обновите». Можно расширить как страховку поверх WS.
> *+* вообще не зависит от стабильности WS. *−* задержка до 5с, лишние запросы.
> 
> **3. STOMP + RabbitMQ (настоящая гарантия доставки).**
> Заменить `enableSimpleBroker` на `enableStompBrokerRelay` + RabbitMQ (или ActiveMQ). Сообщения в **durable-очереди** пользователя, лежат при разрыве и доставляются после reconnect, есть ACK и redelivery. **НО** гарантия только для **персональных durable-очередей** (`/queue/user-{id}`, user-destinations), а не `/topic` — широковещательный topic в relay так же эфемерен.
> *+* настоящая at-least-once. *−* большая переделка: поднять/админить RabbitMQ, переписать destinations с `/topic/{id}` на user-queue, обработать дубли (at-least-once → возможны повторы), персистентность очередей. Для «обновите страницу» — из пушки по воробьям.
> 
> **4. Своя таблица outbox в БД.**
> Писать каждое уведомление в `notifications(user_id, payload, delivered)`, слать по WS, на reconnect клиент запрашивает недоставленное с момента X. Ручная очередь поверх своей же БД.
> *+* гарантия без нового инфраструктурного компонента. *−* велосипед-очередь, чистка таблицы, отметки delivered.

>[!question]- что выбрать для StrikerStat
> **Вариант 1** (re-fetch при reconnect + `visibilitychange`). Природа уведомлений — «перечитай данные», а не «уникальное событие, которое нельзя потерять». Гарантировать надо **актуальность состояния**, а не доставку каждого сообщения. RabbitMQ — корректное промышленное решение для гарантированной доставки, но серьёзная переделка и здесь избыточно.

---

## Видео-оверлей в OBS (/ws-public) — таймер не запускался на препроде

>[!question]- симптом: таймер оверлея запускается в OBS только на localhost, на препроде нужен ручной Refresh
> OBS Browser Source указывает на страницу `pages/olympic_events/video-overlay.vue`. Таймер запускается **не сам по себе**, а по живому STOMP-пушу: оператор жмёт «Начать раунд» → бэк шлёт `timerControl`/`battleStarted` в `/topic/video-overlay/{eventId}` через WS-эндпоинт **`/ws-public`** → страница в OBS ловит и запускает таймер.
> - **localhost работает:** `/ws-public` идёт напрямую в бэкенд `ws://localhost:6300` (без nginx/TLS/edge).
> - **препрод не работает:** `wss://домен/ws-public` через nginx + edge Timeweb + Basic Auth.
> - **почему Refresh помогает:** при перезагрузке `onMounted → loadVideoOverlay()` (REST) читает СОХРАНЁННОЕ состояние таймера и восстанавливает его (`restoreFromSettings`). То есть после Refresh таймер появляется из REST, а не из WS → значит живой WS-пуш до OBS не доходил.
> 
> **OBS как «браузер» тут ни при чём** — это CEF (Chromium), исполняет тот же JS и открывает тот же WebSocket, никакого особого подхода для STOMP-подписки не требует. Проблема в транспорте до препрода.

>[!question]- диагностика по слоям (что выяснили)
> 1. **Браузер (DevTools → Network → WS → `ws-public`):** «Provisional headers are shown», **нет Status Code** (ни 101, ни 200/401), бесконечный reconnect-цикл. Значит апгрейд не получает валидного HTTP-ответа — соединение рвётся до ответа. Это **не Basic Auth** (был бы 401) и **не баг фронта** (заголовки запроса корректные: `Upgrade: websocket`, `Sec-WebSocket-Key`, `Origin`).
> 2. **Бэкенд напрямую (с сервера):** `websocat -v ws://127.0.0.1:6300/ws-public` → `101 Switching Protocols` + `Upgrade: websocket` + `Sec-WebSocket-Accept`. Бэкенд исправен, авторизации `/ws-public` не требует.
> 3. **nginx-конфиг:** `nginx -T 2>/dev/null | awk '/# configuration file/{f=$0} /location \/ws|auth_basic|server_name/{print f": "$0}'` → виновник `/etc/nginx/sites-enabled/front.conf` (симлинк на `sites-available/front.conf`). В нём `server_name strikerstat-preprod.twc1.net`, `auth_basic "Restricted Area"`, есть `location /ws`, но **нет `location /ws-public`**.
> 
> **Вывод:** виноват слой nginx — `/ws-public` либо не маршрутизируется на бэкенд как WS, либо режется server-level `auth_basic` (браузерный `new WebSocket()` не несёт Basic-Auth из URL-кредов надёжно). Бэкенд и фронт исправны.
> 
> NB: `grep -rln "location /ws" /etc/nginx/sites-enabled/` дал пусто, потому что в `sites-enabled` симлинки, а `grep -r` их по умолчанию не разворачивает. Надёжнее искать через `nginx -T` (привязан к реальным путям).

>[!question]- фикс nginx: добавить location /ws-public
> В `server`-блок `front.conf` (где `auth_basic` и `location /ws`) добавить:
> ```nginx
> location /ws-public {
>     auth_basic off;                       # оверлей публичный + WS из браузера не несёт URL-креды
>     proxy_pass http://127.0.0.1:6300;
>     proxy_http_version 1.1;
>     proxy_set_header Upgrade $http_upgrade;
>     proxy_set_header Connection 'upgrade';
>     proxy_set_header Host $host;
>     proxy_set_header X-Real-IP $remote_addr;
>     proxy_read_timeout 3600s;             # против «тихого» обрыва долгого WS
>     proxy_send_timeout 3600s;
> }
> ```
> Безопасная вставка (правим цель симлинка `sites-available/front.conf`):
> ```bash
> cp /etc/nginx/sites-available/front.conf /root/front.conf.bak-ДАТА     # бэкап
> cat > /root/ws-public.block <<'EOF'                                    # 'EOF' в кавычках!
> 
>     location /ws-public {
>         auth_basic off;
>         proxy_pass http://127.0.0.1:6300;
>         proxy_http_version 1.1;
>         proxy_set_header Upgrade $http_upgrade;
>         proxy_set_header Connection 'upgrade';
>         proxy_set_header Host $host;
>         proxy_set_header X-Real-IP $remote_addr;
>         proxy_read_timeout 3600s;
>         proxy_send_timeout 3600s;
>     }
> EOF
> cp /etc/nginx/sites-available/front.conf /tmp/front.conf.new
> sed -i '/auth_basic_user_file/r /root/ws-public.block' /tmp/front.conf.new   # вставка после server-level строки
> diff /etc/nginx/sites-available/front.conf /tmp/front.conf.new               # проверить, что добавилось 1 место
> cp /tmp/front.conf.new /etc/nginx/sites-available/front.conf                 # применить
> nginx -t && nginx -s reload                                                  # reload только если -t OK
> websocat -v 'wss://strikerstat-preprod.twc1.net/ws-public'                   # ждём 101
> ```
> Откат: `cp /root/front.conf.bak-ДАТА /etc/nginx/sites-available/front.conf && nginx -t && nginx -s reload`.
> 
> Тонкость порядка `location`: nginx выбирает по **самому длинному префиксу**, не по порядку записи. `/ws-public` — более длинный префикс, чем `/ws`, поэтому свой блок перехватит запрос независимо от места вставки.
> 
> Вдогонку: в существующий `location /ws` тоже стоит добавить `auth_basic off;` — он защищён cookie-токеном на уровне приложения (`WebSocketHandshakeInterceptor`), Basic Auth там лишний и ломает судейский оверлей так же.

>[!question]- прикладной фикс: re-fetch при (пере)подключении WS
> В `connectOverlaySocket()` (video-overlay.vue), в колбэке успешного `connect`, добавлено `void loadVideoOverlay(); void loadFightList();` ПЕРЕД `subscribe`. SimpleBroker не переотправляет пуши, отправленные пока сокет был не подключён (первый коннект в OBS, обрыв edge/nginx, фоновая вкладка) — теперь при любом коннекте/реконнекте оверлей сам подтягивает актуальное состояние из REST, и пропущенный пуш больше не требует ручного Refresh. Это та же стратегия «re-fetch при reconnect», что в разделе про гарантию доставки.

>[!question]- безопасность правок nginx (откат)
> - `nginx -t` проверяет конфиг, **НЕ применяя** его. Битый синтаксис → ошибка, рабочий nginx продолжает на старом конфиге. Никогда не делать `reload`, если `-t` не зелёный.
> - `nginx -s reload` — graceful: старые воркеры дослуживают, новые встают на новом конфиге; невалидный конфиг просто не применится, падения сайта нет.
> - Перед правкой — бэкап файла (`cp … /root/…bak-ДАТА`). Откат = вернуть бэкап + `nginx -t && nginx -s reload`.
> - Полный слепок активного конфига: `nginx -T > /root/nginx-full-ДАТА.txt 2>&1`.
> - Падение возможно только при `restart`/`stop` с битым конфигом (не при `reload`): тогда `nginx -t` покажет строку ошибки → восстановить бэкап → `systemctl restart nginx`.

>[!question]- почему `auth_basic off` в одиночку не спас (satisfy any + allow/deny)
> Препрод-`server` (443) устроен так:
> ```nginx
> satisfy any;
> allow 83.218.196.40; allow 37.201.116.45; allow 95.26.43.55; allow 178.69.179.253;
> deny all;
> auth_basic "Restricted Area";
> auth_basic_user_file /etc/nginx/.htpasswd;
> ```
> `satisfy any` = доступ даётся, если прошёл **ЛИБО по IP** (allow-список), **ЛИБО по паролю** (auth_basic). Директивы `satisfy`, `allow/deny`, `auth_basic` **наследуются** в locations, если не переопределены.
> - Страница грузится, т.к. браузер шлёт Basic Auth на обычный GET.
> - WebSocket-рукопожатие **Basic Auth не несёт** (заголовки на WS из JS не задать) → если IP не в allow-списке, WS-путь не подтверждается ничем → доступ закрыт → обрыв без HTTP-статуса.
> 
> Поэтому `auth_basic off;` в `location /ws-public` при не-разрешённом IP даёт **403**: auth убрали, а `deny all` из наследованного access-модуля остался, и `satisfy any` больше нечем удовлетворить. Нужен ещё и доступ по access-модулю → `allow all;` (публично) **или** IP клиента в общем `allow`.

>[!question]- два способа открыть `/ws-public` (публично vs по IP)
> **A. Публично (не зависит от адреса клиента):**
> ```nginx
> location /ws-public {
>     allow all;
>     auth_basic off;
>     # + proxy_pass 6300, http_version 1.1, Upgrade/Connection, X-Forwarded-*, timeouts 3600s
> }
> ```
> Эндпоинт становится доступен любому, кто знает `eventId`. Данные там несекретные (имена/счёт/таймер для трансляции), бэкенд `/ws-public` авторизации не требует по дизайну → приемлемо. Остальной стенд остаётся под IP/паролем.
> 
> **B. Закрыто по IP (наш выбор на препроде — статический IP `213.191.26.29`):**
> 1. добавить `allow 213.191.26.29;` в общий allow-список server (перед `deny all;`);
> 2. в `location /ws-public` — только `auth_basic off;` (БЕЗ `allow all;`), тогда он наследует IP-список.
> Твой IP в allow → `satisfy any` пускает по IP без пароля, WS проходит по IP. Бонус: с этого IP весь стенд открывается без ввода пароля.
> Проверять WS надо **с клиентской машины** (`websocat -v 'wss://домен/ws-public'` → 101), а не с сервера — там source-IP = localhost.

>[!question]- IP-allow годится только для СТАТИЧЕСКОГО адреса (динамический — сломается)
> `allow <IP>` привязывает доступ к конкретному адресу. При **динамическом** IP провайдер его периодически меняет (переподключение роутера, аренда DHCP, обрыв). После смены:
> - адрес не совпадёт с `allow` → по IP не пустит;
> - страница снова спросит пароль, по паролю зайдёшь;
> - но **WS опять отвалится** (пароль в рукопожатие не идёт).
> Оверлей ломался бы «сам по себе» при каждой смене IP.
> 
> Правило: **IP-allow — только для статики.** Для динамического IP WS-доступ нельзя вешать на IP — надо либо публичный эндпоинт (вариант A), либо авторизация, которая ходит ВНУТРИ WebSocket (cookie/токен на уровне приложения, как у `/ws` через `WebSocketHandshakeInterceptor`, а не Basic Auth на nginx). Грубый паллиатив — разрешить подсеть провайдера `allow 213.191.26.0/24;` (открывает всю подсеть, ненадёжно).

>[!question]- нужен ли `location /ws-public` на ПРОДЕ
> Скорее всего **нет**. `location /ws` — префиксное совпадение, ловит все URI на `/ws`, включая `/ws-public` (nginx берёт самый длинный префикс, регэкспов нет). Значит `/ws-public` и так проксируется через существующий `location /ws` на `:6300` с апгрейд-заголовками. На препроде ломала **не маршрутизация, а авторизация** (satisfy any + IP + Basic Auth). На проде, где «все адреса доступны» (нет IP-фильтра и Basic Auth), этого блокатора нет.
> 
> **Сначала проверить, ничего не меняя:** `websocat -v 'wss://ПРОД_ДОМЕН/ws-public'` →
> - `101` + `Upgrade: websocket` → уже работает, блок не нужен;
> - не `101` → `location /ws` на проде задан иначе (`location = /ws` / регэксп) и `/ws-public` под него не попал → добавить блок (без `auth_basic off`/`allow all` — там нечего снимать, только proxy+upgrade+timeouts).
> 
> **Вдогонку (даже при 101):** проверить, есть ли у `location /ws` строка `proxy_read_timeout`. Если нет — nginx по умолчанию рвёт «тихое» соединение через **60с**. Для долгого WS оверлея дописать `proxy_read_timeout 3600s; proxy_send_timeout 3600s;` в `/ws` или в отдельный `/ws-public`.


  1. Найти файл с препрод-фронтовым блоком:                                                                                                                                                                                                                                                                         
  grep -rln "location /ws" /etc/nginx/sites-enabled/ /etc/nginx/conf.d/ 2>/dev/null                                                                                                                      
  2. Открыть его и в ТОТ ЖЕ server { … } (где location /ws и auth_basic), рядом с location /ws, добавить:                                                                                                                                                                                                           
  location /ws-public {                                                                                                                                                                                                                                                                                             
      auth_basic off;                                                                                                                                                                                                                                                                                               
      proxy_pass http://127.0.0.1:6300;                                                                                                                                                                                                                                                                             
      proxy_http_version 1.1;                                                                                                                                                                                                                                                                                       
      proxy_set_header Upgrade $http_upgrade;                                                                                                                                                                                                                                                                       
      proxy_set_header Connection 'upgrade';                                                                                                                                                                                                                                                                        
      proxy_set_header Host $host;                                                                                                                                                                                                                                                                                  
      proxy_set_header X-Real-IP $remote_addr;                                                                                                                                                                                                                                                                      
      proxy_read_timeout 3600s;                                                                                                                                                                                                                                                                                     
      proxy_send_timeout 3600s;                                                                                                                                                                                                                                                                                     
  }                                                                                                                                                                                                                                                                                                                 
  3. Проверить конфиг и перезагрузить (только если nginx -t = OK):                                                                                                                                                                                                                                                  
  nginx -t && nginx -s reload                                                                                                                                                                                                               
  4. Проверить апгрейд через домен (должно быть 101, без креды):                                                                                                                                                                                                                                                    
  websocat -v 'wss://strikerstat-preprod.twc1.net/ws-public'                                                                                                                                                                                                                                                        
  Ждём в ответе Upgrade: websocket + Sec-WebSocket-Accept. Если 101 — победа.                                                                                                                                                                                                                                       
  5. В OBS обновить Browser Source (Refresh один раз). Дальше таймер будет запускаться сам по живому пушу, без ручных рефрешей (плюс подстраховка — добавленный re-fetch при реконнекте).

>[!question]- почему Basic Auth (логин/пароль) НЕ работает для браузерного WebSocket, а для GET работает
> Тонкость в **браузерном WebSocket API**, а не в «сокетах вообще». HTTP-запросы и WS-хендшейк аутентифицируются по-разному.
> - **HTTP (страница, `/_nuxt`, `/api`):** после первого ввода пароля браузер кэширует Basic Auth для origin и **сам добавляет** заголовок `Authorization: Basic <base64>` во все `fetch`/XHR/`<script>`/`<img>` → nginx `auth_basic` доволен → 200.
> - **WebSocket:** этот заголовок в хендшейк **не подставляется**, и обойти это в браузере нельзя:
>   1. **API не даёт задать заголовки** — `new WebSocket(url, protocols)` принимает только URL и сабпротоколы, места под `Authorization` нет (в отличие от `fetch` с `headers`).
>   2. **Кэш Basic Auth на WS не распространяется** — браузеры (особенно Chromium/CEF, а OBS = CEF) не прикрепляют закэшированные креды к `Upgrade`-запросу → хендшейк уходит без `Authorization` → nginx `401` → обрыв. Диалог пароля на WS-хендшейк браузер **не показывает**, вручную не спасти.
>   3. **Креды в URL запрещены** — `new WebSocket("wss://user:pass@домен/ws")` браузер отклоняет с `SyntaxError` (userinfo в WS-URL запрещён спекой). Трюк `user:pass@host`, работающий для страницы, для сокета не проходит.
>   → WS-хендшейк физически приходит к nginx **без авторизации**, поэтому `auth_basic` его режет. Отсюда `/ws-public` с `auth_basic off`.
> **Что на WS-хендшейк ВСЁ-ТАКИ едет (для контраста):**
> - **Cookies** — куки того же origin браузер к WS-хендшейку прикрепляет → cookie/сессионная авторизация с WebSocket работает (в отличие от header-based Basic Auth);
> - **query-string / сабпротокол** — токен через `wss://домен/ws?token=…` или `Sec-WebSocket-Protocol`. Так делает наш `/ws`: авторизация не через nginx Basic Auth, а на бэке через `WebSocketHandshakeInterceptor` (токен/сессия в STOMP-CONNECT); `/ws-public` не требует и этого.
> **Суть:** Basic Auth — это HTTP-**заголовок**, а браузерный WebSocket прикрепить его не может (ни через API, ни через кэш, ни через URL). Работают только куки или токен. Поэтому для OBS/`/ws-public` пароль снимаем на прокси (`auth_basic off`).

>[!question]- ЧТО добавить в nginx, чтобы OBS и тестировщик работали БЕЗ завязки на IP
> Почему не по IP: allow-по-IP годится только для статического «своего» адреса. У тестировщика **динамический IP** (протухнет) и/или **VPN со split-tunnel** (подменит адрес — см. блок ## VPN), а **OBS вообще не умеет вводить Basic Auth** (некому нажать «ок» в диалоге 401). Значит доступ надо строить не на IP, а так:
>
> **1) WebSocket оверлея — публичный `location /ws-public`** (браузерный `new WebSocket()` не несёт Basic Auth, а IP OBS произвольный):
> ```nginx
> location /ws-public {
>     auth_basic off;      # снять пароль
>     allow all;           # + снять IP-фильтр (иначе наследованный `deny all` при satisfy any даст 403)
>     proxy_pass http://127.0.0.1:6300;
>     proxy_http_version 1.1;
>     proxy_set_header Upgrade $http_upgrade;
>     proxy_set_header Connection 'upgrade';
>     proxy_set_header Host $host;
>     proxy_set_header X-Real-IP $remote_addr;
>     proxy_cache_bypass $http_upgrade;
>     proxy_read_timeout 3600s;   # долгое соединение не рвать (дефолт 60с)
>     proxy_send_timeout 3600s;
> }
> ```
> Нужны **обе** строки `auth_basic off` + `allow all`: `satisfy any` = «пройти ИЛИ по паролю, ИЛИ по IP»; убрав пароль, надо открыть и IP-ветку, иначе `deny all` из server-блока вернёт 403.
>
> **2) Страница оверлея + ассеты + API — по паролю через URL (nginx менять НЕ надо).** OBS грузит HTML/JS/`_nuxt`/`api` обычными HTTP-GET — их закрывает существующий `satisfy any` + `auth_basic`. Дай OBS Browser Source URL с вшитыми кредами:
> `https://ЮЗЕР:ПАРОЛЬ@домен/olympic_events/video-overlay?eventId=...`
> Браузер закэширует Basic Auth для origin и подставит во все GET (страница, `/_nuxt/*`, `/api/*`) **независимо от IP/VPN**. Только WebSocket так не покрывается — для него и нужен п.1.
>
> **3) Тестировщику для обычной работы в браузере** — просто дай **пароль** (Basic Auth). `satisfy any` пропустит по паролю при любом IP/VPN.
>
> **Итог:** OBS = публичный `/ws-public` (единственная правка nginx) + URL с кредами для страницы; человек = пароль. `allow` по IP оставляем только для статических «своих» адресов. Ничего не завязано на динамический IP.
> Применить: `nginx -t && nginx -s reload`; проверить с клиента: `websocat -v 'wss://домен/ws-public'` → ждём `101`.

---

## Basic Auth по IP: почему на тестинге просил пароль, а через VPN ломается (allow/deny + $remote_addr)

>[!question]- симптом: на новом тестинге `strikerstat-test.twc1.net` при заходе на сайт всё равно спрашивает пароль
> Хотя в 443-`server` стоит `satisfy any; allow 213.191.26.29; … deny all; auth_basic "Restricted Area";`, а `213.191.26.29` — мой статический IP. По этой конфигурации пароль спрашиваться **не должен**. `nginx -t` = OK, reload прошёл. Значит nginx видит от меня **не тот** IP, что вписан в `allow`.

>[!question]- ключевой принцип: `allow/deny` сравнивается с `$remote_addr` — IP TCP-сокета, а не с тем, что «я вписал»
> `$remote_addr` — это адрес, с которого пришло **TCP-соединение** (L3-адрес сокета), а НЕ HTTP-заголовок. Любая прослойка между тобой и nginx (VPN, CDN, балансировщик) подменяет этот адрес на свой. Если он не в `allow` → `satisfy any` не находит разрешённый адрес → падаем в `auth_basic` → пароль.
> **Диагностика — одной строкой:** первое поле в `access.log` = ровно то, что nginx видит в `$remote_addr`. Сравнить с `curl -s https://api.ipify.org` (мой реальный egress-IP). Не совпало → между нами прослойка.
> - Логи vhost'а могут быть переопределены: `access_log /var/log/nginx/front.access.log;` → дефолтный `/var/log/nginx/access.log` **пустой**, смотреть надо `front.access.log`.
> - Проверить, что правка реально в РАБОЧЕМ конфиге: `nginx -T 2>/dev/null | grep -n "213.191.26.29"` (не тот файл / симлинк `sites-enabled` → другой источник — частая ловушка).

>[!question]- что оказалось: VPN в браузере подменял IP (`104.28.202.179` вместо `213.191.26.29`)
> В `front.access.log`: `104.28.202.179 - striker … "GET /sign-in"` — это (1) **не мой** IP и (2) юзер `striker` = зашёл по паролю. `104.28.x` — диапазон Cloudflare, т.е. VPN = **Cloudflare WARP**, выходящий с Cloudflare-адресов.
> Развязка — два `curl` дали **разные** адреса:
> - `curl -s https://api.ipify.org` → `213.191.26.29` (терминал шёл напрямую),
> - а nginx для запроса из Chrome записал `104.28.202.179`.
> Раз у `curl` и браузера разный egress → VPN гонит в тоннель **не весь трафик, а выборочно по адресу назначения** (split tunneling): IP сайта заворачивается в тоннель (→ WARP-IP), а адрес, куда стучался `curl` (ipify), в тоннель не попал (→ реальный IP). Это НЕ «VPN только в браузере» — позже подтвердил таблицей маршрутов (`ip route get`), см. блок **## VPN** ниже.
> **Фикс:** выключить VPN (или исключить IP сайта из тоннеля) → браузер пойдёт с `213.191.26.29`, который в `allow` теста → пускает без пароля. С nginx всё было в порядке.

>[!question]- важно: это НЕ Cloudflare-перед-сайтом (проверка `curl -sI`)
> Сначала подумал, что тестинг стоит за Cloudflare (тогда nginx видел бы CF-edge и нужен был бы `real_ip`). Проверка опровергла: `curl -sI https://strikerstat-test.twc1.net/sign-in | grep -iE 'server|cf-ray'` → `server: nginx/1.24.0 (Ubuntu)`, **без `cf-ray`**. Значит CDN в цепочке перед сайтом нет — сайт обслуживает nginx напрямую. `104.28.x` был **клиентским** VPN (WARP на моей стороне), а не reverse-proxy. Поэтому конфиг с `set_real_ip_from`/`CF-Connecting-IP` НЕ понадобился.
> Мораль: `server: cloudflare` + `cf-ray` в ответе ⇒ сайт проксируется через CF (нужен real_ip); `server: nginx` без `cf-ray` ⇒ прямой доступ, а «чужой» IP в логе — это VPN/прокси на стороне клиента.

---

## VPN: подмена IP (SNAT) и split tunneling

>[!question]- что такое NAT (расшифровка с нуля)
> **NAT = Network Address Translation** — «трансляция сетевых адресов». Это когда устройство на пути (роутер, VPN-нода) **переписывает IP-адреса прямо в заголовке пролетающего пакета**.
> **Зачем придумали:** публичных IPv4-адресов мало, а устройств много. Дома у тебя телефон, ноут, ТВ — у каждого свой **приватный** адрес (`192.168.0.x`), но такие адреса в интернете не маршрутизируются. Провайдер выдаёт один **публичный** IP. NAT позволяет всем внутренним устройствам ходить в интернет через этот один публичный адрес.
> **Как работает (домашний роутер):**
> 1. Телефон шлёт пакет `src=192.168.0.5:51000 → dst=сайт:443`.
> 2. Роутер переписывает источник на свой публичный: `src=95.10.20.30:40000 → dst=сайт:443`, и запоминает пару в **таблице трансляций**: `192.168.0.5:51000 ↔ 95.10.20.30:40000`.
> 3. Сайт видит `95.10.20.30` (не знает про телефон вообще) и отвечает на него.
> 4. Ответ приходит на роутер (`95.10.20.30:40000`), он по таблице делает **обратную замену** `dst → 192.168.0.5:51000` и отдаёт телефону.
> **Два вида (по тому, какой адрес меняется):**
> - **SNAT (Source NAT)** — меняет **источник** у исходящих пакетов. Это наш случай: роутер/VPN прячет отправителя за своим IP. Разновидность SNAT с динамическим внешним IP называют **masquerade** (маскарад).
> - **DNAT (Destination NAT)** — меняет **назначение** у входящих (проброс портов: `внешний_ip:8080 → внутренний_сервер:80`).
> **Связь с VPN:** VPN-нода делает ровно тот же SNAT, что и домашний роутер, только она стоит в другой сети/стране — поэтому сайт видит её IP, а твой реальный спрятан. Отсюда и «подмена IP» в VPN — это просто SNAT на выходной ноде.

>[!question]- как роутер понимает, кому вернуть пакет (телефон vs ноут за одним внешним IP) — порты, PAT/NAPT
> По **порту**. Ключ в таблице трансляций — не один IP, а пара **IP:порт**. При переписывании источника роутер выдаёт каждому соединению **свой уникальный внешний порт**:
> ```
> телефон  192.168.0.5:51000  →  95.10.20.30:40000
> ноутбук  192.168.0.8:51000  →  95.10.20.30:40001   ← тот же внешний IP, но ДРУГОЙ порт
> ```
> Даже если оба устройства взяли одинаковый внутренний порт `51000`, внешние порты разные → внешняя пара `IP:порт` уникальна. Ответ приходит на конкретный порт:
> - на `95.10.20.30:40000` → таблица: «телефон `192.168.0.5:51000`» → отдать телефону;
> - на `95.10.20.30:40001` → «ноутбук» → отдать ноутбуку.
> Порт — это и есть «обратный адрес», по которому роутер узнаёт получателя.
> **Как называется:** эта форма (много хостов за одним IP через трансляцию портов) — **PAT (Port Address Translation) = NAPT**, в народе «NAT overload» / **masquerade**. Чистый NAT (1:1, только IP) так не умеет — делить один адрес позволяет именно трансляция портов.
> Полный ключ в таблице — **5-tuple**: `протокол + src_ip + src_port + dst_ip + dst_port` (поэтому даже один внешний порт можно разрулить по адресу удалённого сайта; но на практике роутер просто раздаёт уникальные внешние порты).
> То же самое делает и VPN-нода: сотни клиентов за одним её публичным IP различаются по порту.

>[!question]- как называется механизм подмены IP в VPN и как он работает
> Это **NAT**, точнее **Source NAT (SNAT) / masquerade** на выходной ноде VPN. «Подмена адреса» = переписывание поля `src` в IP-пакете на выходе.
> Пошагово:
> 1. Твой пакет `src=213.191.26.29 → dst=сайт` шифруется и по тоннелю (интерфейс `tun0`) уходит на VPN-сервер (exit-ноду).
> 2. Exit-нода **переписывает адрес источника**: `src` становится её собственным публичным IP (`104.28.202.179`), и уже так пакет уходит в интернет. Сайт видит IP ноды, а не твой — вот и «подмена».
> 3. Нода запоминает соответствие в таблице трансляций (conntrack): `(твой внутр. IP:порт) ↔ (IP ноды:порт)`.
> 4. Ответ приходит на IP ноды, она по таблице делает **обратную трансляцию** (`dst` → твой IP) и отдаёт пакет тебе через тоннель.
> Это тот же механизм, что и домашний роутер (много устройств LAN → один публичный IP провайдера) — «masquerading»; в VPN выходная точка не роутер провайдера, а нода в другой сети/стране.
> **Важно:** SNAT меняет **только L3-адрес источника** (тот самый `$remote_addr`, что видит nginx). Прикладные заголовки (HTTP, Basic Auth) не трогаются — поэтому закэшированный браузером пароль проходит независимо от того, через VPN ты или нет.

>[!question]- split tunneling: почему часть трафика идёт через VPN, а часть напрямую
> **Split tunneling** = в тоннель заворачивается не весь трафик, а выборочно по адресу назначения. У меня реализовано через **policy-routing** в Linux:
> - отдельная таблица маршрутов `table 2022` с большим списком конкретных CIDR → `dev tun0` (что тоннелировать);
> - правило `ip rule`: `from all lookup 2022 suppress_prefixlength 0` — «смотри в VPN-таблицу, но **игнорируй в ней `default` (/0)**». Итог: адрес уходит в тоннель, только если для него есть **конкретный** префикс в списке; нет накрывающего префикса → проваливается в `main` → напрямую через физический интерфейс.
> **Наш случай (VPN включён на обоих):**
> - `preprod 176.124.215.240` → в списке НЕТ накрывающего префикса (соседний `176.124.176.0/21` = до `.183`, а `.215` в дыре) → `main` → `enp6s0` **напрямую** → сервер видит реальный `213.191.26.29`.
> - `test 94.241.171.236` → попадает в тоннельный префикс → `tun0` (table 2022) → SNAT на ноде → сервер видит `104.28.202.179`.
> Оба Timeweb, «российские» диапазоны, но список префиксов VPN с дырами, и блок препрода попал в дыру. Похоже на VPN, который выборочно заворачивает диапазоны (гео/обход), а не всё подряд.

>[!question]- как быстро проверить, пойдёт ли адрес через VPN или напрямую
> `ip route get <IP>` — ядро показывает фактическое решение маршрутизации:
> ```
> ip route get 94.241.171.236   # → via 172.19.0.2 dev tun0 table 2022   = через VPN (SNAT)
> ip route get 176.124.215.240  # → via 192.168.0.1 dev enp6s0           = напрямую (реальный IP)
> ```
> Вспомогательное:
> - `ip rule show` — правила policy-routing (какие таблицы и в каком порядке смотрятся);
> - `ip route show table 2022` — что именно заворачивается в тоннель;
> - `ip -brief addr | grep tun` — есть ли VPN-интерфейс (`tun0`);
> - `getent hosts <домен>` — резолв домена в IP, когда нет `dig`.
> Со стороны сервера тот же факт виден в первом поле `access.log` (реальный IP vs IP ноды).

>[!question]- как это связывает обе загадки (какой IP видит сервер × пускает ли он)
> Два **независимых** фактора:
> 1. **какой src-IP доходит до сервера** — решает split-tunnel маршрутизация на моей машине (+ SNAT на ноде, если адрес тоннелируется);
> 2. **пускает или просит пароль** — решает `allow`-лист сервера + `satisfy any` (+ закэшированный Basic Auth).
> Сводка:
> - **препрод + VPN** → видит реальный `213.191.26.29` (его IP не тоннелируется), но этого IP **нет в allow препрода** → пароль. Раньше «пускало» = браузер слал кэш Basic Auth из `https://user:pass@`-URL (вторая ветка `satisfy any`, IP не смотрится);
> - **тест без VPN** → реальный `213.191.26.29`, он **в allow теста** (я добавил) → пускает по IP;
> - **тест + VPN** → `104.28.202.179` (тоннель + SNAT), не в allow → пароль.
> Проверено: `curl -sI` обоих → `server: nginx`, без `cf-ray` (Cloudflare ни при чём); allow-лист препрода (`nginx -T`) моего IP не содержит; `ip route get` показал разную маршрутизацию. Практический вывод: **IP-allow хрупок** — любой VPN/прокси/CDN между тобой и origin подменяет `$remote_addr`. Для честной проверки IP-фильтра: без VPN + чистый браузер без сохранённых кредов.

---

## SSE vs WebSocket для оверлея (вернуться позже)

>[!question]- SSE vs WebSocket для оверлея: когда что и стоит ли менять
> Механизм «сервер сам шлёт уведомления клиенту» называется **SSE (Server-Sent Events)**, браузерный API `EventSource`. Это **односторонний** (server → client) стрим по одному длинному HTTP-ответу (`Content-Type: text/event-stream`, события вида `data: {...}\n\n`, опционально `id:`). Родственники: long-polling (старее), HTTP/2 push (мёртв).
>
> **Почему для оверлея трансляции SSE подходит лучше WebSocket:**
> 1. **Двунаправленность не нужна.** Оверлей — чистый broadcast server→client, а действия клиента (старт таймера, смена настроек) **уже идут обычными REST POST** (`/api/events/video-overlay/*`). WS-канал используется только на приём → форма задачи = SSE.
> 2. **Нет проблем с авторизацией на стендах.** SSE — обычный HTTP-GET, браузер **сам шлёт Basic Auth и куки** (в отличие от браузерного WebSocket — см. колаут «почему Basic Auth не работает для WS»). → не нужен `/ws-public` с `auth_basic off`, не нужна возня с IP-allow/VPN, OBS грузил бы поток по тем же кредам, что и страницу.
> 3. **Проще через nginx** — нет `Upgrade`-хендшейка, обычный `proxy_pass` + `proxy_buffering off;` + большой `proxy_read_timeout`.
> 4. **Авто-reconnect встроен** в `EventSource` (не надо ручного reconnect, как у webstomp), плюс **replay по `Last-Event-ID`** — прямой ответ на «SimpleBroker теряет сообщения при разрыве» (см. [[#Гарантия доставки (вернуться позже)]]).
>
> **Чем платишь:**
> - только одна сторона (но клиент→сервер и так REST); только текст/UTF-8 (шлём JSON — ок); лимит соединений HTTP/1.1 ~6 на домен (но у нас `http2` → не проблема);
> - **переписывание messaging-слоя**: STOMP `SimpleBroker` + `convertAndSend` → реестр `SseEmitter` по `eventId` (держать список подписчиков события, при broadcast писать во все emitters). Концептуально проще (нет STOMP/брокера), но это работа → оформлять как **спайк/оценку**;
> - replay по `Last-Event-ID` требует серверного буфера истории — сам `SseEmitter` его не хранит, реализовать вручную.
>
> **Дешёвый средний путь (без большого рефактора):** **SockJS** поверх текущего STOMP — сам падает на fallback-транспорты (eventsource/xhr-streaming/long-polling) когда WS недоступен, а они = обычный HTTP (с Basic Auth/куками). Даёт устойчивость там, где WS режется, не меняя брокер.
>
> **Scope:** предлагать SSE **только для оверлея** (push + REST-действия). Судейские уведомления тоже на WS, но там другая модель — не трогать в этом же заходе.
>
> **Итог:** для broadcast-оверлея SSE — более точное совпадение по форме (односторонний push) и снимает класс проблем с авторизацией на тест-стендах; цена — переписать messaging на `SseEmitter`. Минимальный вариант — включить SockJS-fallback.                                                                                                                           
  