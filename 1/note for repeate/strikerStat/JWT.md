# JWT в StrikerStat (backend-java)

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