# JWT в StrikerStat (backend-java)
Сайты на которых можно генерировать токены https://www.base64decode.org/  https://jwt.io/ 

>[!question]- Какая JWT-библиотека используется?
>Никакая внешняя. JWT реализован вручную в `dto/auth/JwtToken.java`:
>- Jackson — JSON хедера/payload
>- Guava `Hashing.hmacSha256` — подпись
>- `Base64.getUrlEncoder().withoutPadding()` — base64url
>
>Алгоритм всегда `HS256` (`{"alg":"HS256","typ":"JWT"}`). В `pom.xml` нет ни jjwt, ни java-jwt, ни nimbus-jose-jwt.

>[!question]- Где хранится секрет и как он ротируется?
>В `application.properties`:
>```
>app.jwt.current_secret=...
>app.jwt.old_secrets=secret1,secret2
>```
>В `AuthService` они инжектятся через `@Value`. При парсинге сначала пробуется `current_secret`, потом по очереди старые из `old_secrets`. Это позволяет менять ключ, не разлогинивая всех — старые токены остаются валидны до `exp`.

>[!question]- Что лежит в payload обычного токена?
>`{id, sub: userId, role, iat, exp}`. Время жизни — 86400 сек (24 часа), константа `AuthService.expiresInSeconds`.

>[!question]- Как токен попадает в браузер?
>В виде HttpOnly cookie `strikerstat_token`. Параметры (в новом пути через `AuthSessionService.buildSessionCookies`):
>- `HttpOnly=true`, `Secure` зависит от `X-Forwarded-Proto`, `SameSite=Lax`, `Path=/`, `maxAge=86400`
>- В старом пути (`AuthService.authenticateUser`) — `SameSite=Strict`
>
>Параллельно ставятся вспомогательные не-HttpOnly cookies для UI:
>- `login` — userId (читает JS)
>- `accstatus` — роль
>- `strikerstat_impersonating=1` — флаг «зашёл под пользователем»
>
>Также есть legacy cookie `token` — поддерживается на переходный период.

>[!question]- Как сервер проверяет токен на каждом запросе?
>Через `TokenAuthenticationFilter` (extends `GenericFilterBean`), подключённый в `SecurityConfig`:
>```java
>.addFilterBefore(tokenAuthenticationFilter, UsernamePasswordAuthenticationFilter.class)
>```
>Шаги фильтра:
>1. Достать токен: cookie `strikerstat_token` → legacy `token` → `Authorization: Bearer ...`
>2. `authService.parseJwtToken` → `parseAuth` (проверка подписи current и old секретами + парсинг payload)
>3. `userTokenPresenceService.touchByRawToken(loginId, rawToken)` — проверка whitelist и `UPDATE last_presence_on_site_at`
>4. Если `jwtToken.mustBeSwitchedToOriginal()` → `restoreOriginalSession` (возврат из impersonate)
>5. В `SecurityContext` положить `RememberMeAuthenticationToken(token, auth, authorities)`

>[!question]- Что такое token whitelist и зачем он, если JWT stateless?
>Stateless JWT нельзя отозвать до `exp` — это его классическая дыра. В проекте её закрыли: SHA-256 от каждого выданного токена пишется в БД (`user_tokens`: `site_user_id`, `token = sha256(jwt)`, `user_agent`, `ip`, `created_at`, `expired_at`, `last_presence_on_site_at`).
>
>На каждом запросе `touchByRawToken` делает `UPDATE … WHERE site_user_id = ? AND token = sha256(?)`. Если вернулось 0 строк — токен отозван, `SecurityContextHolder.clearContext()`.
>
>Logout = `DELETE FROM user_tokens WHERE token = sha256(...)`.

>[!question]- Как устроен класс JwtToken?
>Это единственное место, где живёт работа с JWT.
>
>**Создание** — `createToken(userId, role, expiresInSeconds, secret)`:
>```
>header  = base64url(json({"alg":"HS256","typ":"JWT"}))
>payload = base64url(json({id, sub, role, iat, exp}))
>sig     = base64url(hmacSha256(secret, header + "." + payload))
>return header + "." + payload + "." + sig
>```
>
>**Парсинг** — конструктор `JwtToken(token, containsBearer, secret, oldSecrets)`:
>1. Снимает префикс `Bearer ` если нужно
>2. Делит по точкам, base64url-декодит
>3. Сверяет подпись через `String.equals` (НЕ константное время — теоретический timing side-channel)
>4. Если current_secret не подошёл — перебирает oldSecrets
>5. Парсит `exp`, достаёт impersonate-поля: `original_id`, `original_role`, `original_exp`

>[!question]- Что такое canBeUsed() и mustBeSwitchedToOriginal()?
>Два состояния валидности для поддержки impersonation:
>- `canBeUsed()` — «не истёк ИЛИ есть валидная оригинальная сессия». Позволяет полу-протухшему impersonate-токену откатиться к админскому.
>- `mustBeSwitchedToOriginal()` — «текущий истёк, но `original_exp` ещё жив». Триггер для авто-возврата к админу прямо в фильтре.

>[!question]- Как работает impersonation (вход под пользователем)?
>1. Админ дёргает endpoint в `AdminPanelUsersController`.
>2. `AuthSessionService.issueImpersonationSession(adminId, rawToken, targetUserId, targetRole, lifetime, req)` создаёт токен с payload:
>   ```
>   {id: target, role: target, exp, original_id: admin, original_role, original_exp}
>   ```
>3. Этот токен заменяет cookie. Все запросы идут под target-пользователем.
>4. Когда impersonate-`exp` истекает, но `original_exp` ещё жив, `TokenAuthenticationFilter` вызывает `restoreOriginalSession` — создаёт новый токен с `{id: original_id, role: original_role, exp: original_exp}`. Проверяет, что оригинальный токен ещё в whitelist.
>
>Кнопка «вернуться в админа» в UI бьёт на `PublicAuthController.restoreOriginalSession` принудительно.

>[!question]- Как устроен токен сброса пароля?
>Отдельный JWT с другим payload — `{uid, exp, hash}`, где `hash = user.getHashForPasswordChange(exp)` (зависит от текущего хэша пароля).
>
>Создание — `AuthSessionService.createPasswordResetToken(user, lifetime)`. Валидация — `validatePasswordResetToken` + `extractUserIdFromPasswordResetToken`.
>
>Хитрость: как только пользователь сменил пароль, `hash` в БД меняется → токен из ссылки больше не сходится → ссылка автоматически инвалидируется.

>[!question]- Как JWT работает с WebSocket?
>На handshake `/ws` стоит `WebSocketHandshakeInterceptor.beforeHandshake`:
>- Читает cookie `strikerstat_token` из `ServerHttpRequest`
>- `parseJwtToken` + `parseAuth` + `touchByRawToken`
>- Если `mustBeSwitchedToOriginal` → `restoreOriginalSession` + Set-Cookie в handshake-ответ
>- `Auth` кладётся в `attributes.put("auth", auth)` — доступно во всей STOMP-сессии
>- Любой fail → `return false`, соединение не открывается (нельзя подключиться через wscat без cookie)
>
>На `/ws-public` интерцептора нет — он только для видео-оверлея и не требует логина.

>[!question]- Как контроллеры получают текущего пользователя?
>Через `BaseController`, от которого наследуются все контроллеры:
>```java
>protected Auth getAuth() {
>    Authentication a = SecurityContextHolder.getContext().getAuthentication();
>    return (Auth) a.getPrincipal();   // принципал положил TokenAuthenticationFilter
>}
>protected Integer getLoginId() { return getAuth().getLoginId(); }
>```
>`Auth` — простой DTO `{loginId, role, isAuthenticated, expiration}`.
>
>Авторизация по роли — через Spring Security authorities: `SimpleGrantedAuthority(role.name())` + `@PreAuthorize("hasAuthority('admin')")` на методах.

>[!question]- Жизненный цикл одного запроса от логина до контроллера?
>1. **Логин** `/api/auth/login` → проверка пароля → `issueSession` → подпись JWT → sha256(JWT) пишется в `user_tokens` → `Set-Cookie: strikerstat_token=...; HttpOnly`.
>2. **REST-запрос** → браузер прицепляет cookie → `TokenAuthenticationFilter` парсит JWT (current/old секреты) → UPDATE `user_tokens` для presence → подставляет `Auth` в `SecurityContext`.
>3. **Контроллер** → `getAuth().getLoginId()` или `@PreAuthorize`.
>4. **Logout** → `revokeSession(userId, rawToken)` → `DELETE FROM user_tokens` → следующая проверка вернёт false.

>[!question]- Какие особенности и риски видны в коде?
>- **Свой JWT** вместо jjwt — экономия зависимостей, но `String.equals` для подписи (не константное время).
>- **Stateful через user_tokens** — токены реально отзываемые, нестандартно для JWT, но закрывает дыру утёкших токенов.
>- **Ротация секретов** через `app.jwt.old_secrets` встроена в парсер.
>- **Impersonation** прозрачно зашит в payload (`original_*`), фильтр сам переключает обратно.
>- **Password reset** — `hash`-поле зависит от текущего пароля → ссылка инвалидируется сменой пароля.
>- Метод `JwtToken.epochMillisToLocaldatetime` назван «millis», но внутри `ofEpochSecond` — название врёт, поведение корректное.
>- В `application.properties` секреты лежат открытым текстом — они скомпрометированы фактом коммита, поэтому ротация и встроена.

>[!question]- Какие ключевые файлы посмотреть?
>- `dto/auth/JwtToken.java` — создание/парсинг/проверка подписи
>- `config/TokenAuthenticationFilter.java` — REST-фильтр
>- `config/websocket/WebSocketHandshakeInterceptor.java` — WS-хендшейк
>- `config/SecurityConfig.java` — подключение фильтра
>- `service/auth/AuthService.java` — старый путь логина, парсер
>- `service/auth/AuthSessionService.java` — issue/revoke/impersonate/restore/password reset, `buildSessionCookies`
>- `service/auth/UserTokenStoreService.java` — sha256 whitelist (`create`/`delete`/`findClientInfo`)
>- `service/auth/UserTokenPresenceService.java` — `touchByRawToken`
>- `controller/BaseController.java` — `getAuth()` / `getLoginId()` для всех контроллеров
>- `service/auth/AuthFlowService.java` — оркестрация login/logout/refresh/password-reset

---

# HTTP-транспорт токена (Bearer, base64, cookie vs header, лимиты)

>[!question]- base64 — это шифрование? Зашифрован ли JWT?
>**Нет.** base64 — это **кодирование**, а не шифрование. Раскодировать может кто угодно без ключа:
>```bash
>echo "eyJpZCI6NCwicm9sZSI6Im9yZ2FuaXplciIsImV4cCI6MTc4OTQ3MjY2MH0" | base64 -d
># {"id":4,"role":"organizer","exp":1789472660}
>```
>JWT состоит из 3 частей через точку: `header.payload.signature`.
>- header и payload — **открытый** base64url-текст, читается всеми;
>- signature — HMAC-SHA256 по секрету.
>
>Подпись защищает **не от чтения, а от подмены**: прочитать `role:organizer` может любой, а поменять на `role:admin` — нет (без секрета не пересчитать подпись).
>
>➡️ **Вывод: в JWT нельзя класть секреты** (пароли, ключи) — они лежат фактически открыто.

>[!question]- base64 vs base64url — в чём разница?
>В обычном base64 есть символы `+`, `/` и паддинг `=`. В **base64url** они заменены: `+`→`-`, `/`→`_`, паддинг убран. Нужно, чтобы токен безопасно жил в URL и заголовках. В подписи видны `-` и `_` — это признак base64url. Поэтому в `JwtToken` отдельный `encodeBase64Url`, а не стандартный base64.

>[!question]- Почему в `Authorization` нужен `Bearer` и почему он так называется?
>`Bearer` = «**предъявитель**», «на предъявителя» (как ценная бумага на предъявителя: кто держит — тот и владелец). Смысл: **кто предъявил валидный токен — того и пускают**, логин/пароль повторно не спрашивают. Поэтому токен берегут как наличные.
>
>`Bearer` — это **название схемы аутентификации**, а не часть токена. Формат заголовка по RFC 7235:
>```
>Authorization: <схема> <учётные-данные>
>```
>Слово `Bearer` говорит серверу: «дальше идёт токен-на-предъявителя, парси как токен». Bearer-токены описаны в **RFC 6750**.

>[!question]- Какие бывают схемы в заголовке Authorization?
>| Схема | Что внутри |
>|---|---|
>| `Basic` | base64(`логин:пароль`) |
>| `Bearer` | токен на предъявителя (JWT, OAuth) — **наш случай** |
>| `Digest` | хеш-челлендж, пароль не передаётся открыто |
>| `Negotiate`/`NTLM` | Kerberos / Windows-аутентификация |
>| `AWS4-HMAC-SHA256` | подпись запроса ключами AWS |
>
>Список расширяемый. `TokenAuthenticationFilter` умеет ровно одну схему — `Bearer` (метод `stripBearerPrefix`, `substring(7)` = длина `"Bearer "`).

>[!question]- Basic — там обязательно логин:пароль?
>**Да, строго** (RFC 7617): `base64(userid + ":" + password)`.
>```bash
>echo "dXNlcjpwYXNz" | base64 -d   # → user:pass
>```
>Это не «два произвольных параметра», а именно `логин:пароль`: первый `:` — разделитель, логин не может содержать `:`, всё после первого `:` — пароль. Тоже не шифрование → только поверх HTTPS.

>[!question]- Cookie vs заголовок Authorization — в чём разница?
>| | Cookie (`strikerstat_token`) | `Authorization: Bearer ...` |
>|---|---|---|
>| Кто шлёт | браузер **автоматически** на каждый запрос | надо добавлять **вручную** |
>| Префикс схемы | нет, голый токен | есть (`Bearer `) |
>| CSRF-риск | **есть** (шлётся автоматом) | нет |
>| Применение | браузер, серверный рендеринг | SPA, мобилки, межсервисные вызовы |
>
>В `TokenAuthenticationFilter` поддержаны оба, **приоритет у cookie**:
>```java
>String token = extractTokenFromCookies(httpRequest);  // 1) cookie (без Bearer)
>if (token == null) {
>    token = extractTokenFromHeader(httpRequest);       // 2) заголовок (с Bearer)
>    containsBearer = true;                              // флаг ТОЛЬКО для заголовка
>}
>```
>Суть: cookie — «браузер сам помнит и предъявляет», `Authorization` — «клиент осознанно предъявляет каждый раз». `Bearer` нужен только в заголовке; в cookie кладётся голый токен.

>[!question]- Сколько данных влезает в токен? Разница cookie vs header?
>В самом JWT (RFC 7519) **лимита нет** — ограничивает транспорт и сервер.
>- **Cookie:** ~**4 КБ на одну cookie** (жёсткий потолок браузера, RFC 6265). Больше — браузер просто не сохранит.
>- **Заголовок:** лимит не на сам `Authorization`, а на **ВСЕ заголовки суммарно**. Tomcat (встроен в Spring Boot): `maxHttpHeaderSize = 8 КБ` по умолчанию. В этот бюджет входят `Host`, `User-Agent`, `Cookie` (cookie тоже едет как заголовок!) и `Authorization` — всё вместе.
>
>Настройка в Spring Boot:
>```properties
>server.max-http-request-header-size=16KB
>```
>Неочевидное: cookie шлётся на **каждый** запрос (включая статику css/js/img) → толстый токен раздувает весь трафик. Заголовок ставится только туда, куда сам решил. ➡️ Ещё один довод держать токен маленьким (только id/role/exp), а тяжёлые данные грузить отдельным запросом по id.

>[!question]- Как воспроизвести 431 Request Header Fields Too Large через curl?
>Послать заголовок больше лимита Tomcat (8 КБ):
>```bash
>BIG=$(printf 'A%.0s' {1..20000})
>curl -i -H "Authorization: Bearer $BIG" http://localhost:6300/event/method
># → HTTP/1.1 431 Request Header Fields Too Large   (иногда 400 Bad Request)
>```
>Через cookie — то же самое (cookie едет как заголовок `Cookie:`):
>```bash
>curl -i --cookie "strikerstat_token=$BIG" http://localhost:6300/event/method
>```
>**Важно:** Tomcat режет заголовки на этапе парсинга HTTP — **раньше** Spring Security. До `TokenAuthenticationFilter` (брейкпоинт в `doFilter`) запрос **не дойдёт**. Это показывает, что контейнер сервлетов стоит ВЫШЕ Spring Security в цепочке.
>
>Доказать, что предел задаёт именно Tomcat: поднять `server.max-http-request-header-size=64KB`, перезапустить — те же 20 КБ дадут 200. Убрать — снова 431.

>[!question]- Почему log.debug в фильтре ничего не пишет?
>Уровень логирования по умолчанию в Spring Boot — **INFO**, а `log.debug(...)` ниже порога (`TRACE < DEBUG < INFO < WARN < ERROR`) → молча отбрасывается. Код при этом выполняется (брейкпоинт сработает), просто сообщение не печатается. `log.isDebugEnabled()` вернёт `false`.
>
>Включить:
>```properties
>logging.level.com.example.testlinux=DEBUG
>logging.level.org.springframework.security=DEBUG   # вся цепочка фильтров в консоли
>```
>⚠️ Логировать сам токен (`log.debug("token: " + token)`) в проде нельзя — токен это секрет.

---

# Крипта: кодирование / хеш / HMAC / шифрование (как работает подпись)

>[!question]- Расшифровка названий — почему так называется
>**MAC** = Message Authentication Code — «код аутентификации сообщения»: тег, доказывающий целостность + подлинность (сообщение не меняли и его создал владелец секрета).
>**HMAC** = **H**ash-based **MAC** — MAC, построенный поверх хеш-функции (хеш + секретный ключ). «H» = на основе хеша.
>**SHA** = **S**ecure **H**ash **A**lgorithm (разработан NSA, стандартизован NIST). **SHA-256** → 256 = длина результата в битах (32 байта). Семейство SHA-2: 224/256/384/512.
>**HS256** (имя в JWT/JWA) = **H**MAC + **S**HA-**256**. Аналогично:
>- **RS256** = **R**SA-подпись + **S**HA-256. **RSA** = фамилии авторов **R**ivest–**S**hamir–**A**dleman.
>- **ES256** = **E**CDSA + **S**HA-256. **ECDSA** = **E**lliptic **C**urve **D**igital **S**ignature **A**lgorithm.
>- **PS256** = RSA-**PSS** (**P**robabilistic **S**ignature **S**cheme) + SHA-256.
>- **EdDSA** = **Ed**wards-curve **DSA** (кривые Ed25519/Ed448).
>**base64** = кодирование алфавитом из 64 символов (см. ниже).
>**JWT** = JSON Web Token; **JWS** = JSON Web Signature (правила подписи); **JWA** = JSON Web Algorithms (имена alg: HS/RS/ES…); **JWK/JWKS** = JSON Web Key / Key Set (формат ключа / набор публичных ключей).
>**bcrypt** (в `PasswordHasher`) = от шифра **B**lowfish + crypt — хеш паролей с встроенной солью.

>[!question]- Биты / байты / символы — «256 бит = 32 байта» это сколько символов?
>Это РАЗНЫЕ единицы, не путать:
>- **бит** — один знак `0`/`1` (мельчайший кубик).
>- **байт** = 8 бит.
>- **символы** — появляются, только когда байты КОДИРУЮТ в текст; число зависит от кодировки.
>
>8 бит = 1 байт: `01001101` = число 77 = hex `4D` (2 симв.) = ASCII `M` (1 симв.).
>16 бит = 2 байта: `01001101 01100001` = `4D61` (4 hex-симв.) = `Ma` (2 симв.).
>| биты | байты | hex-символов | ASCII |
>| 8 | 1 | 2 (`4D`) | `M` |
>| 16 | 2 | 4 (`4D61`) | `Ma` |
>| 256 | 32 | 64 | ~32 (непечат.) |
>
>SHA-256 → **256 бит = 32 байта** (размер СЫРЫХ данных). На экране показывают как **64 hex-символа** или ~43 base64url-символа. Т.е. «32» — это БАЙТЫ (размер), «64» — СИМВОЛЫ (как написали). Байт ≠ символ.
>В сети эти 32 байта едут ВНУТРИ payload пакета (бит/байт — единицы размера, а не «размер пакета»; пакет = заголовки + payload, лимит MTU ~1500 байт).

>[!question]- 256 бит хватает закодировать любое значение? (коллизии)
>«Закодировать однозначно и обратимо» — **НЕТ**. Хеш — это не кодирование, а ОТПЕЧАТОК:
>- кодирование (base64) — обратимо, без потерь, размер растёт с входом, исходник вернуть можно;
>- хеш (SHA-256) — **необратим, с потерями**, фиксированные 256 бит при любом входе; исходник НЕ восстановить.
>
>256 бит = 2²⁵⁶ ≈ 1,16·10⁷⁷ значений — огромно, но КОНЕЧНО. Входов бесконечно много → по принципу Дирихле **коллизии неизбежны** (разные входы → один хеш). Пример: 33-байтных входов 2²⁶⁴ > 2²⁵⁶ хешей → часть столкнётся.
>Почему всё равно «хватает»: найти коллизию практически невозможно (2²⁵⁶ не перебрать; для SHA-256 коллизий не находили). Теоретически есть — практически ненаходимы → для подписи/целостности достаточно.
>Вывод: хеш даёт практически уникальный отпечаток фиксированного размера, но НЕ «упаковывает» значение (вернуть нельзя, в отличие от base64).

>[!question]- Почему SHA-256 необратим? Любой ли хеш необратим?
>Необратимость держится на ДВУХ разных вещах:
>**1. Потеря информации (любой сжимающий хеш).** ∞ входов → фиксированные 256 бит = many-to-one → точный исходник вернуть нельзя.
>Игрушка `h(x)=x mod 10`: `h(7)=h(17)=h(27)=7` — по `7` вход не определить, «десятки» выброшены.
>➡️ Любой сжимающий хеш необратим в смысле «не вернуть точный оригинал».
>**2. Односторонность по построению (только крипто-хеши).** У `x mod 10` точный вход не вернуть, но подобрать ЛЮБОЙ тривиально (хоть `7`). SHA-256 добавляет: нельзя найти даже КАКОЙ-ТО вход (preimage resistance) — перебор ~2²⁵⁶.
>| | вернуть ТОЧНЫЙ вход | найти ЛЮБОЙ вход |
>| любой сжимающий хеш | нет (потеря) | у простых — легко |
>| крипто-хеш SHA-256 | нет | нет (2²⁵⁶, невозможно) |
>
>Чем достигается односторонность в SHA-256:
>- операции, теряющие информацию: `AND`/`OR` (`a AND b=0` → входы 00/01/10?), сложение `mod 2³²` (перенос отбрасывается);
>- перемешивание: XOR, сдвиги, повороты битов, 64 раунда → входные биты «размазаны» по всему выходу;
>- лавинный эффект: 1 бит входа → меняется ~половина битов выхода, без связи «кусок входа = кусок выхода».
>Отмотать 64 раунда назад = перебор на каждом шаге → экспоненциальный взрыв.
>**Контраст:** обратимость требует БИЕКЦИИ — шифрование (AES) и кодирование (base64) биективны (вход↔выход, без потерь, есть «расшифровать»/«раскодировать»). Хеш намеренно НЕ биекция: сжимает + перемешивает необратимыми операциями.

>[!question]- 4 разных понятия — не путать (кодирование / хеш / HMAC / шифрование)
>| Операция | Обратима? | Нужен ключ? | Зачем | В JWT |
>| Кодирование (base64) | ✅ кем угодно | ❌ | байты → текст | header, payload, signature |
>| Хеширование (SHA-256) | ❌ | ❌ | отпечаток данных | — |
>| HMAC (хеш + ключ) | ❌ | ✅ секрет | подпись / подлинность | signature |
>| Шифрование | ✅ с ключом | ✅ | спрятать содержимое | НЕ используется |
>
>Главное заблуждение: «base64 шифрует». НЕТ. base64 — кодирование (читается без ключа). Защиту даёт HMAC с секретом, не base64.

>[!question]- Чем кодирование отличается от шифрования (и от хеша)
>Оба ОБРАТИМЫ (в отличие от хеша), но цель и ключ разные:
>| | Кодирование (base64) | Шифрование (AES/RSA) |
>| зачем | совместимость/формат (байты→текст) | секретность (спрятать) |
>| нужен ключ | ❌ | ✅ |
>| кто вернёт | кто угодно (алгоритм публичный) | только владелец ключа |
>| защита от чтения | ❌ нет | ✅ да |
>
>Суть: кодирование меняет ПРЕДСТАВЛЕНИЕ (чтобы данные пролезли по каналу), секретности ноль — и это задумано; шифрование меняет данные так, что без ключа НЕ ПРОЧИТАТЬ.
>Пример:
>```
>кодирование: "Man" → base64 → "TWFu"        ← любой обратно (echo|base64 -d)
>шифрование:  "Man" + ключ → AES → "9f3a8c…"  ← вернуть только с ключом
>```
>Тройной контраст:
>- **кодирование** → обратимо БЕЗ ключа (формат)
>- **шифрование** → обратимо С ключом (секрет)
>- **хеш** → необратимо вообще (отпечаток)
>JWT: payload в base64url = кодирование (читается всеми); JWT НЕ шифрует (только подписывает HMAC) → секреты в payload нельзя. Шифрование было бы в JWE (в проекте нет).
>JWT payload НЕ зашифрован → секреты в него класть нельзя.

>[!question]- Что такое base64 (это НЕ «64 символа на выходе»)
>64 — размер АЛФАВИТА, не длина результата. base64 представляет произвольные байты алфавитом из 64 печатных символов: `A–Z`(0–25), `a–z`(26–51), `0–9`(52–61), `+`(62), `/`(63).
>Почему 64: один символ кодирует 6 бит (2⁶=64). Алгоритм: берём 3 байта = 24 бита → режем на 4 группы по 6 бит → каждое число 0–63 → символ алфавита. **3 байта → 4 символа** (раздувание ~33%).
>Пример "Man":
>```
>M=77       a=97       n=110
>01001101   01100001   01101110   ← 24 бита
>010011 010110 000101 101110      ← по 6 бит
>19     22     5      46
>T      W      F      u           → "TWFu"
>```
>Не кратно 3 байтам → паддинг `=`. Полностью обратимо, без ключа.
>base64url (в коде `Base64.getUrlEncoder().withoutPadding()`): `+`→`-`, `/`→`_`, `=` убран — т.к. эти символы спецсимволы в URL/заголовках. Признак base64url в подписи: `-` и `_` (`_Z7nEOAmay-...`).

>[!question]- Как генерируется подпись (generateBase64UrlEncodedSignature) — два шага
>```java
>byte[] guavaHash = Hashing.hmacSha256(secret.getBytes(UTF_8)).hashString(message, UTF_8).asBytes(); // (1) HMAC = безопасность
>return encodeBase64Url(guavaHash);                                                                    // (2) base64url = просто текст
>```
>1. **HMAC-SHA256** — вот это «подпись». Только владелец `secret` посчитает правильные байты. (HMAC = хеш + секретный ключ; обычный SHA-256 без ключа подделал бы кто угодно.)
>2. **base64url** — HMAC даёт сырые байты (0–255, непечатаемые) → кодируем в текст для токена. НЕ про безопасность.
>`message` = `headerEncoded + "." + payloadEncoded` (подписываются header и payload вместе) — см. ниже «JWS Signing Input».

>[!question]- message = headerEnc.payloadEnc — это правило HMAC или JWT? Можно ли свою формулу?
>Это правило **JWT/JWS (RFC 7515)**, НЕ HMAC. Называется **JWS Signing Input**: `ASCII( BASE64URL(header) + "." + BASE64URL(payload) )`.
>HMAC к формату не причастен — он **message-agnostic**: подпишет любые байты (хоть header.payload, хоть картинку). Принцип HMAC — «замешать секрет в хеш сообщения», ЧТО за сообщение — не его дело.
>- «Всегда ли header.payload?» → да, ДЛЯ JWT (правило JWS), но не универсально.
>- «Это принцип HMAC?» → нет, надстройка JWT.
>
>Почему именно такой формат (не «своя формула»):
>1. Подписывают header И payload (не только payload) — header содержит `alg`; иначе атакующий поменяет `alg` (none / RS↔HS) и подпись не заметит. Подпись «привязывает» алгоритм.
>2. Разделитель `.` — однозначность («ab»+«cd» vs «abc»+«d» = «abcd» без разделителя неотличимы). Точки нет в алфавите base64url → безопасный разделитель.
>3. Сначала base64url, потом склейка — части это JSON (могут содержать `.{}`, не-ASCII); base64url → безопасный алфавит без разделителей.
>
>Почему НЕ побитовые операции / своя хитрая сборка:
>- Криптостойкость — в HMAC (СЕКРЕТЕ), а НЕ в способе сборки message. Сборка должна лишь: быть детерминированной (обе стороны байт-в-байт одинаково, иначе подписи не сойдутся), однозначной (без коллизий: 2 разных header/payload → разный message), полной.
>- Хитрая формула = 0 доп. безопасности (атакующего останавливает незнание секрета, а не «секретность сборки») + риск коллизий/багов. Принцип Керкгоффса: стойкость на КЛЮЧЕ, не на сокрытии алгоритма → формат публичный.
>- Интероперабельность: токен с Java проверяется на Python/Go/Node, только если ВСЕ формируют signing input идентично. Своя формула → проверит лишь твой код → смысл стандарта потерян.

>[!question]- Как проверяется подпись (validateToken) — это ПЕРЕСЧЁТ, а не расшифровка
>Сервер НЕ расшифровывает подпись. Он заново вычисляет ожидаемую и сравнивает:
>```
>expected = base64url( HMAC_SHA256(secret, headerEnc + "." + payloadEnc) )
>valid = expected.equals(signatureEnc из токена)
>```
>- поменяли payload (role→admin) → expected не совпадёт → невалиден;
>- подделать подпись без секрета нельзя (не посчитать HMAC-байты).
>Метод `generateBase64UrlEncodedSignature` СОЗДАЁТ подпись; ПРОВЕРКУ делает `validateToken`, вызывая его и сравнивая результат со строкой из токена.
>В strikerstat при неудаче с current_secret перебираются old_secrets (ротация ключей).

>[!question]- HS256 симметричный + ловушка .equals
>**HS256 = симметричный**: один `secret` и подписывает, и проверяет (в отличие от RS256: приватный ключ подписывает, публичный проверяет). Поэтому секрет живёт ТОЛЬКО на сервере — у кого секрет, тот кует любые токены.
>⚠️ Сравнение `signatureExpected.equals(signatureCurrent)` — НЕ constant-time → теоретический timing side-channel. «Правильно» — `MessageDigest.isEqual` / константное сравнение.

>[!question]- Разбор по полочкам: Hashing.hmacSha256(secret...).hashString(message...).asBytes()
>```java
>byte[] guavaHash = Hashing.hmacSha256(secret.getBytes(UTF_8)).hashString(message, UTF_8).asBytes();
>```
>Цепочка Guava по шагам:
>1. `secret.getBytes(UTF_8)` — секрет (строка из application.properties) → байты ключа.
>2. `Hashing.hmacSha256(keyBytes)` — создаёт `HashFunction`, **настроенный на HMAC-SHA256 с этим ключом**. Ключ уже «вшит» в функцию.
>3. `.hashString(message, UTF_8)` — прогоняет `message` (= `headerEnc + "." + payloadEnc`) через HMAC → объект `HashCode`.
>4. `.asBytes()` — достаёт результат как `byte[]` (32 байта = 256 бит, длина SHA-256).
>Итог: `guavaHash` = СЫРЫЕ байты подписи (MAC). Дальше `encodeBase64Url(guavaHash)` → текст для токена.
>Входов у HMAC два (ключ + сообщение), выход один (32 байта). Повторяемо только с тем же ключом.

>[!question]- Где в коде HS256 и почему это то же, что hmacSha256
>HS256 фигурирует в ДВУХ местах под разными именами:
>- header (createToken): `headerMap.put("alg", "HS256")` — что ЗАЯВЛЯЕМ в `{"alg":"HS256",...}`.
>- вычисление (generateBase64UrlEncodedSignature): `Hashing.hmacSha256(...)` — РЕАЛИЗАЦИЯ.
>`HS256` = JWA-сокращение: **HS** = HMAC-SHA, **256** = длина. То есть HS256 ≡ HMAC-SHA256.
>⚠️ Проект НЕ читает `alg` из токена при проверке (всегда HMAC-SHA256 жёстко в коде) — это хорошо: защита от атаки подмены `alg` (напр. `alg:none` или RS↔HS confusion).

>[!question]- HMAC — это НЕ способ создания ключа
>Частое заблуждение. HMAC ключ НЕ создаёт. Ключ (секрет) уже существует (`app.jwt.current_secret`). HMAC его ИСПОЛЬЗУЕТ.
>HMAC = Hash-based Message Authentication Code: берёт 2 входа (секрет + сообщение) → 1 выход (MAC). Это «хеш, замешанный на секрете»: обычный SHA-256 посчитал бы кто угодно (без ключа) → подделка; HMAC даёт правильный результат только владельцу секрета.
>Внутри (концептуально, делает Guava): `HMAC(K,m) = SHA256( (K⊕opad) ‖ SHA256( (K⊕ipad) ‖ m ) )` — двойной хеш с двумя константами (защита от length-extension; наивный `SHA256(K‖m)` уязвим).
>Терминология: HMAC даёт **MAC** (симметрично, один секрет), а не цифровую подпись (RS256/ES256 — асимметрично).

# Алгоритмы подписи: HS vs RS/ES (симметрия vs асимметрия)

>[!question]- JWT поддерживает разные alg — какие
>Поле `alg` в header определяет алгоритм:
>| alg | что | тип |
>| HS256/384/512 | HMAC+SHA | симметрично (один секрет) |
>| RS256/384/512 | RSA+SHA | асимметрично (приватный/публичный) |
>| ES256/384/512 | ECDSA+SHA | асимметрично (эллиптич. кривые) |
>| PS256… | RSA-PSS | асимметрично |
>| EdDSA | Ed25519 | асимметрично |
>| none | без подписи | ⚠️ опасно, запрещать |
>Проект жёстко использует HS256.

>[!question]- Как меняется ПРОВЕРКА при RS256/ES256 (вместо hmacSha256)
>Концептуальная разница: «проверить» значит РАЗНОЕ.
>- HS256 (симметрия): проверка = заново вычислить подпись секретом + сравнить строки (`expected.equals(actual)`). Проверяющий МОЖЕТ воспроизвести подпись.
>- RS256/ES256 (асимметрия): воспроизвести подпись НЕЛЬЗЯ (нет приватного ключа). Вместо пересчёта — математическая верификация ПУБЛИЧНЫМ ключом, сразу boolean. `equals` исчезает.
>
>RS256-проверка (java.security вместо Guava):
>```java
>byte[] signatureBytes = BASE64URL_DECODER.decode(signatureEncodedFromToken); // подпись из токена → байты
>Signature verifier = Signature.getInstance("SHA256withRSA");                 // RS256
>verifier.initVerify(publicKey);                                              // ПУБЛИЧНЫЙ ключ, не секрет
>verifier.update(message.getBytes(UTF_8));                                    // header.payload
>boolean valid = verifier.verify(signatureBytes);                            // сразу true/false
>```
>ES256 — то же, но `Signature.getInstance("SHA256withECDSA")` + EC-ключ. ⚠️ JWT хранит ECDSA-подпись как сырое `R‖S` (64 байта), а Java ждёт DER → нужна конвертация (в Nimbus спрятана). У RSA такого нет.
>Подписывает (на логине) держатель приватного ключа: `initSign(privateKey) … sign()`.
>Итог: строка `Hashing.hmacSha256(...).asBytes()` + `equals` → заменяются на `Signature...verify(publicKey)`.

>[!question]- Когда HS256, когда RS256/ES256 (дело НЕ в числе репозиториев)
>Критерий — границы доверия и разделение «кто выпускает ↔ кто проверяет», а НЕ моно/мульти-репозиторий.
>- **HS256** — когда один доверенный сервис и выпускает, и проверяет (как strikerstat). Секрет никуда не уходит. Проще и быстрее.
>- **RS256/ES256** — когда токены проверяют стороны, которым НЕЛЬЗЯ доверить выпуск / нельзя отдать секрет: отдельный auth-сервер (issuer) + много resource-сервисов (только verify), или внешние потребители. Публичный ключ раздаётся безопасно (часто через JWKS `/.well-known/jwks.json`), приватный — только у issuer.
>Решающий вопрос: «должен ли проверяющий уметь выпускать?» нет → асимметрия.
>Контрпримеры: микросервисы на общем секрете через secrets-manager = HS256 ок (но компрометация любого = форж за всех); один сервис, но токены проверяют внешние = нужна асимметрия, хоть «проект один».
>На практике распределённые системы/микросервисы обычно тянут к асимметрии — но из-за границ доверия, не из-за количества репозиториев.

>[!question]- Секрет JWT — это соль, перец или что?
>Ни то, ни другое. «Соль» и «перец» — термины ХЕШИРОВАНИЯ ПАРОЛЕЙ; секрет JWT — это **ключ HMAC** (подпись/проверка сообщения), другой механизм.
>| | что | секретна? | где | уникальна? |
>| соль (salt) | случайное на КАЖДЫЙ пароль | нет | рядом с хешем | да, на запись |
>| перец (pepper) | секрет, общий для всех | да | отдельно (код/конфиг/HSM) | нет, один на всех |
>| секрет JWT | ключ HMAC (signing key) | да | код/конфиг | один на всех |
>
>- **salt** к JWT не относится: соль — в хешировании паролей, случайная, на каждый пароль, хранится открыто (bcrypt в `PasswordHasher` зашивает соль прямо в хеш).
>- **pepper** ближе по духу (тоже «секрет в коде/конфиге, а не в БД»), но перец — добавка к хешу ПАРОЛЯ, а секрет JWT — ключ ПОДПИСИ токена. Совпадает только «секрет вне БД», назначение разное.
>Правильное название: **секретный ключ HMAC (signing key)**, не соль и формально не перец.
