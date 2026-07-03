# NGINX
установка сертифката 
https://sky.pro/wiki/html/kak-nastroit-https-na-vashem-sajte/ 

>[!question]- что такое nginx (в двух словах)
> **nginx** — высокопроизводительный **веб-сервер** и **reverse-proxy** (обратный прокси). Обычно стоит «на входе» перед приложением и делает:
> - **reverse-proxy** — принимает запросы снаружи и раздаёт их внутренним сервисам (`proxy_pass http://127.0.0.1:6300` и т.п.);
> - **TLS-терминация** — держит HTTPS-сертификат, расшифровывает трафик, а к бэкенду ходит по голому HTTP на `127.0.0.1`;
> - **единый вход (один домен/порт 443)** — по `location` разводит: `/` → фронт, `/ws` → бэкенд, `/api` → бэкенд… Один origin → нет CORS, наружу торчит только nginx;
> - **раздача статики** (`root`/`alias`), gzip, кэш;
> - **балансировка** между несколькими бэкендами (`upstream`), лимиты, access-фильтры (`allow`/`deny`, `auth_basic`).
> 
> Модель: **событийная, асинхронная** (не поток-на-соединение), поэтому держит десятки тысяч соединений дёшево — важно для долгих WebSocket-коннектов.
> 
> Конфиг: главный файл `/etc/nginx/nginx.conf`, он через `include` подтягивает сайты из `/etc/nginx/sites-enabled/*` (обычно [[LINUX/BASE#Симлинк|симлинки]] на `sites-available/`). Полный собранный конфиг: `nginx -T`.

>[!question]- на каких протоколах работает nginx и как это настраивается (с примерами)
> nginx ловит **не «любой протокол»**, а то, что описано в его блоках. Два разных мира:
> 
> **1. `http {}` — HTTP/HTTPS (L7).** Сюда входят HTTP/1.0, HTTP/1.1, HTTP/2, HTTPS и **WebSocket** (т.к. WS начинается как HTTP-запрос с `Upgrade`).
> ```nginx
> http {
>     server {
>         listen 443 ssl http2;          # HTTPS + HTTP/2
>         server_name example.com;
>         ssl_certificate     /path/fullchain.pem;
>         ssl_certificate_key /path/privkey.pem;
> 
>         location / {                   # обычный HTTP-проксинг
>             proxy_pass http://127.0.0.1:3000;
>         }
>         location /ws {                 # WebSocket = HTTP + Upgrade той же TCP-сессии
>             proxy_pass http://127.0.0.1:6300;
>             proxy_http_version 1.1;
>             proxy_set_header Upgrade $http_upgrade;
>             proxy_set_header Connection 'upgrade';
>             proxy_read_timeout 3600s;  # не рубить «тихий» долгий WS (дефолт 60s)
>         }
>     }
> }
> ```
> **WebSocket — не отдельный протокол на старте:** это HTTP/1.1-запрос с `Upgrade: websocket`, после чего **та же TCP-сессия** переключается в WS-фрейминг. Поэтому WS живёт внутри `http{}`/`location`, а не требует особого блока — нужны лишь `proxy_http_version 1.1` + `Upgrade`/`Connection`, чтобы nginx пропустил апгрейд.
> 
> **2. `stream {}` — сырой TCP/UDP (L4).** Для протоколов, которые НЕ HTTP: БД, SMTP, MQTT, игровые серверы, TCP-проброс, DNS(UDP). Здесь nginx работает как «тупой» TCP/UDP-балансировщик, не понимая содержимое.
> ```nginx
> stream {
>     upstream db { server 127.0.0.1:5432; }
>     server {
>         listen 6432;                   # TCP по умолчанию
>         proxy_pass db;
>     }
>     server {
>         listen 53 udp;                 # UDP (например DNS)
>         proxy_pass 127.0.0.1:5353;
>     }
> }
> ```
> **Итого:** HTTP/HTTPS/HTTP2/WS → `http{}`; произвольный TCP/UDP → `stream{}`. Чего нет ни в одном блоке — nginx не обрабатывает. `http{}` и `stream{}` — соседние секции верхнего уровня в `nginx.conf`.

>[!question]- где в запросе IP-адрес клиента и почему allow/deny работает даже для WebSocket
> IP клиента живёт **НЕ в HTTP-заголовке и не в MIME**, а на **сетевом уровне (L3, заголовок IP-пакета)** — это source-IP TCP-соединения. Его нельзя «не указать»: каждый пакет обязан нести src/dst IP, иначе нет маршрутизации.
> - nginx берёт адрес **из сокета** (peer TCP-соединения) → переменная **`$remote_addr`**. По ней работают `allow`/`deny`.
> - Заголовки `X-Forwarded-For`/`X-Real-IP` — **опциональные**, их добавляют **прокси** (не браузер), чтобы протащить исходный IP дальше; легко подделываются, для фильтрации nginx их НЕ использует. Строка `proxy_set_header X-Real-IP $remote_addr;` — это nginx сам прокидывает socket-IP бэкенду, чтобы Spring видел клиента, а не `127.0.0.1`.
> - **Почему `allow` работает и для WS:** проверка идёт на уровне соединения (socket-IP), а WS использует **ту же TCP-сессию**, что и стартовый HTTP-запрос → тот же `$remote_addr`. Поэтому `allow 213.191.26.29;` действует одинаково на GET и на WS-апгрейд.
> 
> **Каверза с промежуточным прокси/edge (напр. edge Timeweb):** тогда `$remote_addr` = IP этого прокси, а не реального клиента, и `allow` по клиентскому IP сломается. Лечится модулем `real_ip`:
> ```nginx
> set_real_ip_from 10.0.0.0/8;        # доверенная подсеть edge/балансировщика
> real_ip_header   X-Forwarded-For;   # взять реальный клиентский IP отсюда
> ```

>[!question]- как сделать так чтобы он запускался после ребутва автоматически 
> sudo systemctl enable nginx.service

>[!question]- Как преминить новые конфиги без остановки nginx
>nginx -t && nginx -s reload
>nginx -t --- проверяет файлы конфигурации
>`nginx -s reload`, который отправляет сигнализатор **SIGHUP** рабочему процессу. 

>[!question]- смотреть логи живом времени
>```
>tail -f /var/log/nginx/front.access.log /var/log/nginx/front.error.log
>```


>[!question]- как чинить обрывы WebSocket-соединения (STOMP) через nginx (code 1006)
> **Симптом:** в браузере WS-соединение рвётся (`CloseEvent code: 1006`, `wasClean: false`), уведомления перестают приходить. В консоли webstomp — `WebSocket is already in CLOSING or CLOSED state` и растущий `did not receive server activity for the last ...ms`.
> 
> **Причина:** для `location /ws` в nginx по умолчанию `proxy_read_timeout 60s` — nginx рубит "тихое" соединение через 60 секунд. Heartbeat не всегда спасает.
> 
> **Где искать конфиг** (nginx на хосте):
> ```bash
> which nginx                                   # /usr/sbin/nginx
> nginx -T 2>/dev/null | grep -nE "server_name|listen|location|proxy_pass|/ws|6300"
> grep -rnE "proxy_pass|/ws|6300" /etc/nginx/   # покажет файл и строку
> ```
> Файлы сайтов: `/etc/nginx/sites-enabled/` ([[LINUX/BASE#Симлинк|симлинки]] на `/etc/nginx/sites-available/`).
> Браузер ходит на `:443` → это `front.conf`, блок `location /ws`.
> 
> **Что нужно в блоке `location /ws`** (обязательный минимум для WS):
> ```nginx
> location /ws {
>     proxy_pass http://127.0.0.1:6300;
>     proxy_http_version 1.1;                # обязательно для WS
> 
>     proxy_read_timeout  3600s;             # ← ключевое: не рубить долгие коннекты (дефолт 60s)
>     proxy_send_timeout  3600s;             # ←
> 
>     proxy_set_header Host $host;
>     proxy_set_header X-Real-IP $remote_addr;
>     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
>     proxy_set_header X-Forwarded-Proto $scheme;
> 
>     proxy_set_header Upgrade $http_upgrade; # переключение протокола на WS
>     proxy_set_header Connection 'upgrade';
>     proxy_cache_bypass $http_upgrade;
> }
> ```
> 
> **Применить** (без рестарта, живые коннекты не рвёт):
> ```bash
> cp /etc/nginx/sites-available/front.conf /etc/nginx/sites-available/front.conf.bak  # бэкап
> nano /etc/nginx/sites-available/front.conf                                          # правка
> nginx -t            # проверка синтаксиса
> nginx -s reload     # применить
> ```
> 
> **Проверка:** DevTools → Console → включить timestamps (⚙️ «Show timestamps»), смотреть время до `STOMP WebSocket closed`.
> - Живёт минуты, обрывов нет → дело было в 60s-таймауте, готово.
> - Рвётся примерно через то же время → режет не nginx, а **edge-балансировщик Timeweb** (`.twc1.net`). Тогда nginx бессилен — смотреть таймауты балансировщика в панели Timeweb / писать в поддержку про "max WebSocket connection lifetime".

>[!question]- что означают флаги nginx (`-t`, `-T`, `-s ...`) и когда их использовать
> Все команды — это бинарь `/usr/sbin/nginx` с флагом. Запускать от root (или через sudo).
> 
> **`nginx -t`** — **t**est. Проверяет синтаксис конфигурации, **ничего не применяя**.
> - Когда: **всегда перед** `reload`/рестартом, после правки конфига.
> - Вывод: `syntax is ok` + `test is successful` — можно применять.
> ```bash
> nginx -t
> ```
> 
> **`nginx -T`** — test + dump. То же, что `-t`, но дополнительно **выводит всю собранную конфигурацию** в stdout (с раскрытыми `include`).
> - Когда: чтобы понять, что реально загружено, и найти нужный `location`/`server`/`proxy_pass` без копания по файлам.
> ```bash
> nginx -T 2>/dev/null | grep -nE "server_name|location|proxy_pass|/ws"
> ```
> Через docker 
> ```
> docker exec -i nginx-proxy nginx  -T 2>/dev/null | grep -nE "server_name|location|proxy_pass|/ws"     
> ```
> **`nginx -s <signal>`** — **s**ignal. Шлёт сигнал уже **запущенному** мастер-процессу. Возможные сигналы:
> - `nginx -s reload` — перечитать конфиг **без рестарта**; живые соединения не рвутся (старые воркеры дорабатывают). Основной способ применить изменения.
> - `nginx -s stop` — немедленная остановка (жёстко, не дожидаясь запросов).
> - `nginx -s quit` — мягкая остановка (graceful: доработать текущие запросы и выйти).
> - `nginx -s reopen` — переоткрыть лог-файлы (нужно после ротации логов).
> ```bash
> nginx -s reload
> ```
> 
> **Прочие полезные:**
> - `nginx -v` — версия; `nginx -V` — версия + флаги сборки и подключённые модули.
> - `nginx -c /path/conf` — запуск с указанным конфигом (по умолчанию `/etc/nginx/nginx.conf`).
> - `nginx -g "directive;"` — задать глобальную директиву из командной строки.
> 
> **Через systemd (альтернатива `-s`)** — управление nginx как сервисом:
> - `systemctl reload nginx` ≈ `nginx -s reload` (перечитать конфиг).
> - `systemctl restart nginx` — полный рестарт (рвёт соединения, нужен редко).
> - `systemctl status nginx` — статус (running/failed, последние логи).
> - `systemctl enable nginx.service` — автозапуск после ребута (см. вопрос выше).
> 
> **Типовой порядок применения правок:**
> ```bash
> nano /etc/nginx/sites-available/front.conf   # 1. правка
> nginx -t                                      # 2. проверка синтаксиса
> nginx -s reload                               # 3. применить (или systemctl reload nginx)
> ```

>[!question]- как локализовать, КТО рвёт WebSocket: бэкенд / nginx / edge (Timeweb)
> **Идея:** обрыв воспроизводят по слоям, подключаясь к WS на разной "глубине" и засекая время жизни. Где соединение начинает рваться — там и виновник.
> 
> **Симптом для справки:** в браузере WS рвётся стабильно (напр. ~20с), `CloseEvent code: 1006, wasClean: false`. При этом обычные HTTP-запросы того же юзера идут без сбоев минутами (видно в логах бэка по `presence/ping`) → значит проблема строго в долгоживущем WS-соединении, а не в доступности.
> 
> **Важно про ~20с vs 60с:**
> - дефолтный `proxy_read_timeout` nginx = **60с**; если рвётся раньше (20-30с) — это, скорее всего, **НЕ nginx-таймаут**.
> - у Spring есть `setTimeToFirstMessage` (дефолт **60с**): сырой WS без STOMP `CONNECT` бэкенд штатно закроет на 60с — это нормально, а не баг.
> - idle-таймаут сбрасывается активностью; если соединение с heartbeat'ами всё равно рвётся строго по времени — пахнет жёстким лимитом на прослойке (edge), а не таймаутом.
> 
> **Шаг 0. Что где слушает:**
> ```bash
> docker ps                                  # бэкенд-контейнер, порт (напр. 127.0.0.1:6300)
> docker logs -f --tail=50 <container>       # видны HTTP-запросы; WS/STOMP на уровне INFO НЕ логируется
> grep -nE "access_log|error_log" /etc/nginx/sites-available/*.conf /etc/nginx/nginx.conf
> ```
> 
> **Шаг 1. Лог nginx в момент обрыва** (вторая сессия + воспроизвести в браузере):
> ```bash
> tail -f /var/log/nginx/error.log
> tail -f /var/log/nginx/access.log | grep --line-buffered "/ws"
> ```
> - `upstream prematurely closed connection` → закрыл **бэкенд**
> - `upstream timed out` → закрыл **nginx** (таймаут)
> - пусто, а коннект умер → рубит **edge/сеть** перед nginx
> - в access.log у строки `/ws` смотри `$request_time` — фактическую длительность.
> 
> **Шаг 2. Тест по слоям (требует cookie-токен из браузера).**
> Имя куки: `strikerstat_token` (DevTools → Application → Cookies → домен). Без неё handshake-интерцептор отдаёт `200` вместо `101` (защита от анонимных подключений — задумано).
> 
> Быстрая проверка через `websocat` (одна бинарка):
> ```bash
> curl -L -o /usr/local/bin/websocat \
>   https://github.com/vi/websocat/releases/latest/download/websocat.x86_64-unknown-linux-musl \
>   && chmod +x /usr/local/bin/websocat
> # длинная форма --header= ; короткий -H в этой сборке глючит ("No URL specified")
> websocat -v --header='Cookie: strikerstat_token=ТОКЕН' ws://127.0.0.1:6300/ws
> ```
> Успех = в ответе `Upgrade: websocket` + `Sec-WebSocket-Accept` (а не `200`).
> ⚠️ websocat НЕ шлёт STOMP/heartbeat → проверяет только сырой WS (idle). Для точного диагноза нужен heartbeat-скрипт ниже.
> 
> **Шаг 3. Точный тест: реальный STOMP + heartbeat по трём путям** (Python):
> ```bash
> python3 -m pip install --quiet websockets
> cat > wstest.py <<'PY'
> import asyncio, ssl, sys, time, websockets
> URL, TOKEN = sys.argv[1], sys.argv[2]
> CONNECT = "CONNECT\naccept-version:1.2\nheart-beat:10000,10000\n\n\x00"
> async def main():
>     ctx = None
>     if URL.startswith("wss://"):
>         ctx = ssl.create_default_context(); ctx.check_hostname=False; ctx.verify_mode=ssl.CERT_NONE
>     hdr = {"Cookie": f"strikerstat_token={TOKEN}"}; kw = {"ssl": ctx} if ctx else {}
>     try:
>         try: ws = await websockets.connect(URL, extra_headers=hdr, **kw)
>         except TypeError: ws = await websockets.connect(URL, additional_headers=hdr, **kw)
>     except Exception as e:
>         print("CONNECT FAILED:", repr(e)); return
>     t=time.time(); print("[  0s] WS opened"); await ws.send(CONNECT)
>     async def beat():
>         while True:
>             await asyncio.sleep(9); await ws.send("\n")
>             print(f"[{round(time.time()-t):>3}s] >>> heartbeat")
>     bt=asyncio.create_task(beat())
>     try:
>         async for msg in ws:
>             head = msg.split("\n",1)[0] if isinstance(msg,str) else "binary"
>             print(f"[{round(time.time()-t):>3}s] <<< {head!r}")
>     except Exception as e:
>         print(f"[{round(time.time()-t):>3}s] CLOSED: {e!r}")
>     finally:
>         bt.cancel(); print(f"LIFETIME: {round(time.time()-t)}s")
> asyncio.run(main())
> PY
> TOKEN='<вставить strikerstat_token из браузера>'
> python3 wstest.py ws://127.0.0.1:6300/ws "$TOKEN"                       # 1) прямо в бэкенд (мимо nginx+edge)
> python3 wstest.py wss://127.0.0.1:443/ws "$TOKEN"                       # 2) через локальный nginx (мимо edge)
> python3 wstest.py wss://strikerstat-preprod.twc1.net/ws "$TOKEN"        # 3) весь путь, как браузер
> ```
> 
> **Таблица вывода (по LIFETIME):**
> 
> | 1) backend | 2) local nginx | 3) public | Виноват |
> |---|---|---|---|
> | живёт | живёт | рвётся ~20с | **edge Timeweb** (лимит WS на балансировщике → панель Timeweb / поддержка) |
> | живёт | рвётся ~20с | рвётся ~20с | **nginx** (крутить таймауты/`proxy_buffering off` в `location /ws`) |
> | рвётся ~20с | — | — | **бэкенд** (idle-таймаут Tomcat/Spring WS-сессии) |
> 
> **Замечание:** сырое соединение `websocat` к бэкенду, державшееся 5+ минут, уже доказывает, что бэкенд не рубит рано. Если `wss://127.0.0.1:443` упрётся не в тот vhost — он `default_server` на 443, обычно ловит; при проблемах подменить Host/SNI.

>[!summary]- ИТОГ диагностики WS-обрывов на strikerstat (что выяснили)
> **Симптом:** на препроде WS-уведомления (STOMP) рвутся стабильно ~20-30с, `code 1006 wasClean:false`, и так по кругу. Локально такой проблемы НЕТ.
> 
> **Почему локально работает, а на препроде нет:**
> - Локально фронт ходит **напрямую в бэкенд** — `ws://localhost:6300/ws` (и API на `:6300`), **без nginx и без edge**. Прослойки, которая рвёт, в пути просто нет → WS живёт, сообщения доходят.
> - На препроде путь: браузер → `wss://…:443` → **nginx** → (**edge Timeweb?**) → бэкенд `:6300`.
> 
> **Три факта, которые сошлись (бэкенд исправен, рубит прослойка):**
> 
> | Сценарий | Путь | Результат |
> |---|---|---|
> | Локально, браузер | прямо в `:6300` | WS живёт долго ✅ |
> | Препрод, `websocat`/python | прямо в `127.0.0.1:6300` | держался 5+ мин ✅ |
> | Препрод, браузер | через nginx + edge | рвётся ~20с ❌ |
> 
> **Вывод:** виноват слой между браузером и бэкендом, который есть ТОЛЬКО на препроде. nginx-конфиг `/ws` корректный (есть `Upgrade`/`Connection`/`proxy_http_version 1.1`), но **нет `proxy_read_timeout`** (дефолт 60с) — а рвётся на 20с, т.е. это, скорее всего, **edge Timeweb**, а не nginx.
> 
> **План добивания:**
> 1. Добавить в nginx `location /ws`: `proxy_read_timeout 3600s; proxy_send_timeout 3600s;` → снять nginx с подозрений (`nginx -t` → `nginx -s reload`).
> 2. Если браузер всё равно рвёт ~20с → это **edge Timeweb** → панель Timeweb (таймаут/лимит времени жизни WS на балансировщике/анти-DDoS) или их поддержка.
> 3. Проверить наличие edge: `dig +short strikerstat-preprod.twc1.net` vs IP самой VM (`ip -4 addr`); лишние заголовки в `curl -sI https://…`; видит ли nginx реальный client-IP в access.log.
> 
> **Что уже починено в коде (отдельно от обрывов):**
> - Фронт `webSocketService.js`: убрана утечка «зомби»-клиентов и heartbeat-таймеров при reconnect (не затирали `onclose` webstomp → не вызывался его `_cleanUp`; + teardown старого клиента перед новым). Теперь reconnect чистый, обрывы стали некритичными.
> - Бэкенд: инвалидация кэшей (`OLYMPIC_ELIMINATION_BRACKETS` и др.) при WS-уведомлениях через `EventCacheEvictor` (после коммита транзакции) — фронт по уведомлению перечитывает свежие данные.
> 
> **Важно:** обрыв `1006` — это аварийный разрыв TCP. Сообщение `WebSocket is already in CLOSING or CLOSED state` — это НЕ сам обрыв, а попытка клиента писать (heartbeat/DISCONNECT) в уже мёртвый сокет. И `SimpleBroker` Spring НЕ переотправляет сообщения, посланные в момент, когда клиент отключён — поэтому стабильность WS важна для доставки уведомлений.


# To do 
1. **Поиск по части имени**

Если название содержит слово "parking" (например, `my_parking_file.txt`):
sudo find / -type f -name "*parking*" 2>/dev/null


### **1. `user www-data;`**

- **Что делает**: Задаёт пользователя и группу, от имени которых будут работать **worker-процессы Nginx**.
    
- **Почему `www-data`**:
    
    - В Ubuntu/Debian пользователь `www-data` создаётся специально для веб-серверов (Nginx/Apache).
        
    - Это повышает безопасность: процессы не запускаются от `root`.
        

---

### **2. `worker_processes auto;`**

- **Что делает**: Определяет количество **рабочих процессов** Nginx.
    
- **`auto`**: Nginx автоматически выбирает оптимальное число (обычно равно количеству CPU-ядер).
    
- **Можно указать вручную**:
    
    nginx
    
    Copy
    
    worker_processes 4;  # Например, для 4-ядерного CPU.
    

---

### **3. `pid /run/nginx.pid;`**

- **Что делает**: Указывает путь к файлу, где хранится **PID (Process ID)** главного процесса Nginx.
    
- **Зачем нужно**:
    
    - Система и скрипты используют этот файл для управления Nginx (перезагрузка, остановка).
        
    - Пример:
        
        bash
        
        Copy
        
        sudo kill -HUP $(cat /run/nginx.pid)  # Перезагрузка конфига.
        

---

### **4. `error_log /var/log/nginx/error.log;`**

- **Что делает**: Задаёт путь к файлу **логов ошибок**.
    
- **Уровни логирования**:  
    Можно добавить уровень серьезности (например, `error_log /var/log/nginx/error.log warn;`).  
    Доступные уровни: `debug`, `info`, `notice`, `warn`, `error`, `crit`, `alert`, `emerg`.
    

---

### **5. `include /etc/nginx/modules-enabled/*.conf;`**

- **Что делает**: Подключает все конфигурационные файлы из папки `/etc/nginx/modules-enabled/`.
    
- **Зачем**:
    
    - В этой папке обычно лежат [[LINUX/BASE#Симлинк|симлинки]] на модули Nginx (например, `ngx_http_geoip_module.conf`).
        
    - Позволяет включать/отключать модули без правки основного `nginx.conf`.





на  сервере timeweb с ОС ubuntu  - запущен проект на vue/nuxt как узнать где он находится и как с ним взаимодействует nginx 


ls -la /var/www/strikerstat-front/pages/ 
 