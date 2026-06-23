
1. [ ] если хеширование основано на потере данных, то за счет чего достигается уникальность кадого хеша, как предотвращаются коллизии ?
2. [ ] почему используется guava для хеширования  HMAC secret - это такой стандарт?
3. [ ] Что может предложить Spring Security для использования jwt
4. [ ] @PreAuthorize @PostAuthorize - судя по @EnableMethodSecurity эти методы прдназначены дял методов, если так, то в чем их приемущество перед проверкой не в магии антоаций , а в коде? Все равно над каждым методом надо ставить. А потоом @PostAuthorize еще и  порядок написания операций нарушает - пишется над методом и вызывается в конце, тем самым запутывая программиста ? В каком случае такой аспект над методом может быть удобнее чем проверка в коде ?
5. [ ] почему в  хедер и нагрузка складываются - это такой способ - спецификация  JWT
6. [ ] как спринг отправляет токен пользователю после заходя на сайт 
7. [ ] что такое Base64 - как происходит кодирование
8. [ ] какие алгоритмы могут быть у jwt хедера
9. [ ] все ли jwt хранят данные в Base64
10. [ ] сколько в хедере может хранится данных - определяется браузером и бэкендом
11. [ ] в ServletRequest/doFilter находится только хедер ? как получить боди
12. [ ] GenericFilterBean единственный способ перехватить запрос ?
13. [ ] напиши приложение спринг секурити , в котором есть ендпоинт, конфиг, класс обработки  jwt , вспомни что такое crsf, cors 
14. [ ] напиши фронт для посылки запросов с хедором jwt (есть ли в хедере мими типы)
15. [ ] пример с сессиией 
16. [ ]  @AfterReturning(pointcut = "@within(checkEventOwnership)", returning = "result") 
17. [ ] сквозная функциональность - общая для всех методов ?
18. [x] у нас в проекте секрет токена (сигнатура jwt) хранится в коде - это называется не соль а перец? - нет, это HMAC секрет

>[!question]- Hello World Security (минимальный SecurityFilterChain)
>`@EnableWebSecurity` + бин `SecurityFilterChain`. Каждый запрос проходит цепочку фильтров, в конце `AuthorizationFilter` проверяет правила `authorizeHttpRequests`.
>```
>@Configuration
>@EnableWebSecurity
>public class SpringSecurityConfig {
>    @Bean
>    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
>        http
>            .csrf(AbstractHttpConfigurer::disable)
>            .authorizeHttpRequests(authz -> authz
>                .requestMatchers("/demo/**").permitAll()      // публично
>                .requestMatchers("/api/admin/**").hasAuthority("Organizer")
>                .anyRequest().authenticated());               // всё остальное — только с аутентификацией
>        return http.build();
>    }
>}
>```

>[!question]- Почему запрос на эндпоинт даёт 403, хотя я не ставил authenticated()
>В Spring Security 6 если запрос **не совпал ни с одним** `requestMatcher` и нет финального `.anyRequest()...` — доступ **запрещается по умолчанию** (deny).
>403 кидает не мой фильтр, а стандартный `AuthorizationFilter` дальше по цепочке: `chain.doFilter()` просто передаёт запрос дальше, а отказ формирует `AuthorizationFilter` → `AccessDeniedException` → `ExceptionTranslationFilter`.
>Почему **403**, а не **401**: пользователь анонимный, а `formLogin`/`httpBasic` не настроены, поэтому точка входа по умолчанию — `Http403ForbiddenEntryPoint`.
>Решение для теста: добавить путь в whitelist — `.requestMatchers("/demo/**").permitAll()`.

>[!question]- Структура JWT (header / payload / signature)
>Токен = `header.payload.signature`, каждая часть в **base64url**, разделены точками.
>```
>{"alg":"HS256","typ":"JWT"}                          ← header: алгоритм и тип
>{"id":4,"role":"organizer","exp":1789472660}         ← payload (claims): кто ты и до когда валиден
>signature                                            ← подпись
>```
>Подпись = `HMAC_SHA256( base64url(header) + "." + base64url(payload), секрет )`. Подписываются header и payload **вместе**, секретом сервера.
>Поэтому payload нельзя подменить: изменишь `id`/`role` — подпись перестанет сходиться.
>«Мусор» вроде `����&k/...` — это сырые байты подписи; в токене они хранятся в base64url (`_Z7nEOAmay-...`).

>[!question]- Может ли тело/параметры запроса лежать в payload JWT (почему там нет myName)
>Нет. Это два разных слоя:
>- **JWT payload** формируется **один раз при логине** и содержит идентичность (`id`, `role`, `exp`). Токен дальше статичен.
>- **Параметры запроса** (`?myName=Igor`, тело) приходят **на каждый запрос** и разбираются контроллером через `@RequestParam` / `@RequestBody`.
>
>Токен был выпущен задолго до запроса и про `myName` ничего не знает, поэтому его там и не может быть.

>[!question]- Зачем нужен JSESSIONID
>HTTP — протокол **без состояния**: сервер сам не помнит, что два запроса от одного клиента. `JSESSIONID` — это «номерок от гардероба»:
>1. сервер создаёт `HttpSession` (корзинку с данными у себя в памяти);
>2. присваивает ей id и отдаёт клиенту: `Set-Cookie: JSESSIONID=...`;
>3. браузер шлёт cookie обратно на каждый запрос → сервер по id находит ту же корзинку.
>
>В сессии обычно хранят: классическую аутентификацию (form-login), корзину, шаги визарда, CSRF-токен.

>[!question]- Нужен ли JSESSIONID при JWT-аутентификации
>Обычно нет. JWT хранит состояние **у клиента (в токене)**, а сессия — **на сервере**. При JWT фильтр на каждый запрос заново читает токен, проверяет подпись и кладёт `Authentication` в `SecurityContextHolder`, в `HttpSession` не заглядывая.
>Поэтому для чистого JWT ставят:
>```
>http.sessionManagement(s -> s.sessionCreationPolicy(SessionCreationPolicy.STATELESS));
>```
>Тогда сервер вообще не создаёт сессии и не шлёт `JSESSIONID`.
>⚠️ STATELESS безопасен, только если ничто не использует `HttpSession` (OAuth2-login, `@SessionScope`-бины, многошаговые формы).

>[!question]- Почему JSESSIONID создаётся даже на 403 / разный в каждом ответе curl
>**Создаётся всегда** — из-за `SessionCreationPolicy.ALWAYS`: сессия создаётся на **каждый** запрос независимо от результата (даже при 403). Это не связано с самим отказом.
>**Разный каждый раз в curl** — потому что curl по умолчанию не хранит и не отправляет cookie обратно: каждый вызов = «новый клиент без сессии» → сервер создаёт новую сессию → новый `JSESSIONID`.
>Чтобы переиспользовать (как браузер) — cookie jar:
>```
>curl -i -c cookies.txt -b cookies.txt "http://localhost:6300/demo/hello?myName=Igor"
>```

>[!question]- Откуда сообщение "Using generated security password" в логах
>Печатает `UserDetailsServiceAutoConfiguration`. Spring Boot создаёт дефолтного пользователя `user` со случайным паролем, **если в контексте нет ни одного** бина типа `UserDetailsService` / `AuthenticationProvider` / `AuthenticationManager`.
>При JWT этот юзер не нужен. Убрать — отключить автоконфигурацию:
>```
>@SpringBootApplication(exclude = { UserDetailsServiceAutoConfiguration.class })
>```
>Или объявить любой `UserDetailsService` (например `InMemoryUserDetailsManager`) — сообщение исчезнет.


# На фронте 
>[!question]- зарпос на сервер через curl cookie
>```
>curl -i http://localhost:6300/event/method \
> --cookie "strikerstat_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NCwicm9sZSI6Im9yZ2FuaXplciIsImV4cCI6MTc4OTQ3MjY2MH0._Z7nEOAmay-TgkWuv2H0XhmXsl_eUlmGJ_KxCSdT4O8"
>```

>[!question]- зарпос на сервер через curl заголовок
>```
>curl -i http://localhost:6300/event/method \
> -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NCwicm9sZSI6Im9yZ2FuaXplciIsImV4cCI6MTc4OTQ3MjY2MH0._Z7nEOAmay-TgkWuv2H0XhmXsl_eUlmGJ_KxCSdT4O8"
>```


curl -i -X PUT "http://localhost:6300/event/update?eventId=175" \
--cookie "strikerstat_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NCwicm9sZSI6Im9yZ2FuaXplciIsImV4cCI6MTc4OTQ3MjY2MH0._Z7nEOAmay-TgkWuv2H0XhmXsl_eUlmGJ_KxCSdT4O8"


```
curl -i -X PUT "http://localhost:6300/event/update-from-dto" \
    -H "Content-Type: application/json" \
    --cookie "strikerstat_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NCwicm9sZSI6Im9yZ2FuaXplciIsImV4cCI6MTc4OTQ3MjY2MH0._Z7nEOAmay-TgkWuv2H0XhmXsl_eUlmGJ_KxCSdT4O8" \
    -d '{"test": 0, "eventId": 175}'
```

---

# Механика: фильтры, контекст, авторизация по ролям

> HTTP-транспорт токена (Bearer, base64 vs шифрование, схемы Authorization, Basic, cookie vs header, лимиты размеров, 431) вынесен в [[JWT]] → раздел «HTTP-транспорт токена».

>[!question]- filterChain выполняется на каждый запрос?
>**Нет.** `SecurityConfig.filterChain(HttpSecurity)` — это **фабрика конфигурации**: выполняется ОДИН раз при старте и собирает бин `SecurityFilterChain`. На каждый запрос работает не этот метод, а **сами фильтры**, которые он собрал.
>- «Обработка запросов **настраивается** в filterChain» ✅
>- «Обработка запросов **происходит** в filterChain» ❌ — происходит в цепочке фильтров.

>[!question]- Полный путь входящего запроса (кто стоит выше Spring Security)
>```
>HTTP-запрос
>   → Tomcat / Servlet-контейнер        (режет заголовки >8КБ ещё ДО Spring → 431)
>   → DelegatingFilterProxy ("springSecurityFilterChain")   мост контейнер→Spring
>   → FilterChainProxy                  выбирает нужный SecurityFilterChain
>       ├─ CorsFilter
>       ├─ TokenAuthenticationFilter     (наш) — аутентификация: кто ты
>       ├─ ... штатные фильтры ...
>       └─ AuthorizationFilter           применяет authorizeHttpRequests: что можно
>   → DispatcherServlet → контроллер
>```
>Первый «спринговый» класс на запрос — `FilterChainProxy`, а не наш filterChain. Контейнер (Tomcat) стоит ВЫШЕ Spring Security.

>[!question]- Как filterChain «меняет поведение» контроллеров
>Никак не трогает контроллеры. Security — **вахтёр ПЕРЕД контроллером**: проходишь проверку → метод вызывается как обычно; не проходишь → метод **не вызывается вообще** (403/401), брейкпоинт в нём не сработает.
>Проверка: брейкпоинт в защищённом методе + запрос без токена → метод не стартует. Это и есть «изменение поведения» — Security решает, дойдёт ли управление до кода.

>[!question]- Брейкпоинты для дебага аутентификации (TokenAuthenticationFilter)
>1. `HttpServletRequest httpRequest = (HttpServletRequest) request;` — сырой запрос от Tomcat. Evaluate: `httpRequest.getRequestURI()`, `httpRequest.getHeader("Authorization")`, `httpRequest.getCookies()`.
>2. `Auth auth = authService.parseAuth(jwtToken);` — токен превращён в id+роль.
>3. `userTokenPresenceService.touchByRawToken(...)` — проверка токена по БД (whitelist).
>4. `SecurityContextHolder.getContext().setAuthentication(authentication);` — ключевая строка: кладёт пользователя в контекст, откуда его возьмёт AuthorizationFilter.
>Логи цепочки: `logging.level.org.springframework.security=DEBUG`.

>[!question]- Правила authorizeHttpRequests — порядок и типы
>Проверяются СВЕРХУ ВНИЗ, **срабатывает первое совпадение** → узкие правила выше общих.
>- `.permitAll()` — всем, даже без токена
>- `.authenticated()` — любой валидный пользователь
>- `.hasAuthority(Role.X)` — конкретная роль
>Пример ловушки порядка: `/feature-state` permitAll ДОЛЖЕН стоять выше общего `/challenges/**` hasAuthority(Fighter), иначе гость не пройдёт.
>⚠️ В этом проекте цепочка НЕ заканчивается `.anyRequest().denyAll()` → путь без совпавшего правила проходит БЕЗ ограничений (нестандартно, в чистом SS6 дефолт — deny).

>[!question]- hasAuthority vs hasRole — главная ловушка ROLE_
>`hasRole("X")` == `hasAuthority("ROLE_" + "X")` — **молча дописывает префикс ROLE_**.
>```
>hasAuthority("Organizer") → ищет строку "Organizer"
>hasRole("Organizer")      → ищет строку "ROLE_Organizer"
>```
>Фильтр кладёт `new SimpleGrantedAuthority(auth.getRole().name())` → "Organizer" БЕЗ префикса.
>Значит `hasRole("Organizer")` даст **молчаливый 403** (ищет ROLE_Organizer, а в контексте Organizer).
>➡️ В этом проекте ВСЕГДА `hasAuthority` + `Auth.Role.X.name()`, никогда `hasRole`. Касается и `@PreAuthorize("hasRole(...)")`, и `hasAnyRole`.

>[!question]- Как hasAuthority(Auth.Role.Organizer.name()) связан с SecurityContext (типы vs строки)
>SecurityContextHolder НЕ знает про enum `Auth.Role`. Сравниваются **строки**, не типы.
>- `.name()` у enum → строка: `Auth.Role.Organizer.name()` → `"Organizer"`.
>- При старте в правило записывается строка `"Organizer"` (enum исчезает).
>- В фильтре `new SimpleGrantedAuthority(auth.getRole().name())` → тоже строка `"Organizer"` (обёртка над строкой).
>- AuthorizationFilter: `"Organizer".equals("Organizer")` → пускает.
>Обе стороны независимо зовут `.name()` на одном enum → строки совпадают. Берут `.name()` (а не литерал `"Organizer"`) ради защиты от опечаток: переименуешь константу — компилятор заставит поправить оба места.
>⚠️ Auth.Role различает «имя константы» (`name()` → Organizer/Fighter/Judge/Trainer) и «сырые роли» из токена (`roleNames`: Referee, member, sunsey...). Маппинг сырой→константа делает `Role.getRole(...)`. В контекст и правила идёт ТОЛЬКО `.name()` константы. Токен `"Referee"` → `Auth.Role.Judge` → authority `"Judge"`.

>[!question]- principal в SpEL — что это и где связывается с Auth
>`principal` — НЕ класс/пакет, а **свойство корня SpEL** `SecurityExpressionRoot` (`org.springframework.security.access.expression`).
>Цепочка `principal.loginId`:
>```
>SpEL "principal" → SecurityExpressionRoot.getPrincipal()
>   → Authentication.getPrincipal()   (тип Object → потому IDE не видит .loginId)
>   → объект из SecurityContextHolder = RememberMeAuthenticationToken
>   → возвращает поле principal = наш Auth
>   → ".loginId" → auth.getLoginId()
>```
>**Связывание principal ↔ Auth** — в фильтре: `new RememberMeAuthenticationToken(authenticationToken, auth, grantedAuthorities)` — 2-й аргумент (тип `Object principal`) = `auth`. Поэтому IDE ругается «Cannot resolve loginId» (статически Object), а в рантайме работает.

# @PreAuthorize (method-security)

>[!question]- Минимальный рабочий @PreAuthorize
>1. Включить: `@EnableMethodSecurity` на конфиге — БЕЗ него аннотация молча игнорируется.
>2. На методе:
>```java
>@PreAuthorize("hasAuthority('Organizer') and @accessService.doesEventBelongToOrganizer(principal.loginId, #eventId)")
>@PutMapping("/update")
>public ResponseEntity<Void> updateEvent(@RequestParam Integer eventId) { ... }
>```
>SpEL-переменные: `principal` (= Auth), `#eventId` (аргумент по имени), `@accessService` (бин по имени).
>`and` короткозамкнут: если `hasAuthority` = false (аноним), второе условие НЕ вычисляется → ни запроса в БД, ни NPE на `principal.loginId`. Провал → 403 (AccessDeniedException → LoggingAccessDeniedHandler).
>Срабатывает через AOP-прокси бина, ДО тела метода.

>[!question]- Зачем @EnableMethodSecurity и почему @PreAuthorize без него молчит
>**Аннотация сама по себе ничего не делает** — это пассивные метаданные («наклейка»). Должен быть кто-то, кто их читает и действует. `@EnableMethodSecurity` регистрирует этого «кого-то».
>Что он регистрирует (через @Import) — advisor-бины (pointcut+advice):
>- `@PreAuthorize`  → `AuthorizationManagerBeforeMethodInterceptor`
>- `@PostAuthorize` → `AuthorizationManagerAfterMethodInterceptor`
>Авто-прокси-постпроцессор видит advisor → оборачивает бин в прокси → на вызове метода вычисляет SpEL → пускает / кидает AccessDeniedException.
>**Без @EnableMethodSecurity** этих advisor-бинов НЕТ → ни один pointcut не матчится → прокси не создаётся → аннотация инертна. Метод выполняется как без неё — **молча, без ошибки** → тихая дыра (думаешь, защищён, а он открыт; брейкпоинт в проверке не сработает, вернётся 200).
>
>Это общий паттерн Spring «аннотация = метаданные, @Enable* = инфраструктура»:
>| @Transactional → @EnableTransactionManagement | @Cacheable → @EnableCaching | @Scheduled → @EnableScheduling | @Async → @EnableAsync |
>(Часть @Enable* в Boot включена автоконфигом — напр. транзакции; method-security по умолчанию ВЫКЛ → включаешь руками.)
>
>Флаги `@EnableMethodSecurity`: `prePostEnabled=true` (дефолт → @PreAuthorize/@PostAuthorize), `securedEnabled` (@Secured), `jsr250Enabled` (@RolesAllowed). (До SS6 было `@EnableGlobalMethodSecurity(prePostEnabled=true)`.)
>
>⚠️ Это ОТДЕЛЬНЫЙ механизм от URL-правил: `authorizeHttpRequests` работает без @EnableMethodSecurity (часть @EnableWebSecurity, живёт в фильтрах). Method-security — на уровне AOP-прокси бинов, ему нужно своё включение. Поэтому в конфиге стоят ОБЕ: @EnableWebSecurity + @EnableMethodSecurity, они не заменяют друг друга.

>[!question]- @PostAuthorize чем отличается от @PreAuthorize
>Различие — КОГДА срабатывает и ЧТО видит:
>- `@PreAuthorize` — ДО метода, видит аргументы (`#eventId`, `principal`). «Можно ли тебе вызвать метод».
>- `@PostAuthorize` — ПОСЛЕ метода, видит результат через **`returnObject`**. «Можно ли тебе видеть ЭТОТ результат».
>
>| | @PreAuthorize | @PostAuthorize |
>| когда | до тела | после возврата |
>| в SpEL | principal, #аргументы | + `returnObject` |
>| метод при отказе | НЕ выполняется | ВЫПОЛНЯЕТСЯ, результат отбрасывается |
>| кейс | решение по входу (#eventId) | решение по загруженному объекту |
>
>Зачем Post: когда владельца видно ТОЛЬКО загрузив объект.
>```java
>@PreAuthorize("@accessService.doesEventBelongToOrganizer(principal.loginId, #eventId)")
>public Event getEvent(Integer eventId) {...}              // eventId на входе
>@PostAuthorize("returnObject.organizerLogin == principal.loginId")
>public Event findBySlug(String slug) {...}                // владелец виден лишь в результате
>```
>⚠️ ГЛАВНАЯ ЛОВУШКА: @PostAuthorize СНАЧАЛА выполняет метод → все побочные эффекты уже случились (запись в БД, письмо, внешний вызов). @Transactional откатит БД при AccessDeniedException, но письмо/HTTP откатить нельзя.
>➡️ @PostAuthorize — ТОЛЬКО на читающих методах (GET/load). На мутациях — никогда, только @PreAuthorize.
>Нюансы: `returnObject` может быть null → property navigation бросит → `returnObject?.organizerLogin`. Для коллекций — НЕ @PostAuthorize, а @PostFilter (фильтрует элементы результата) / @PreFilter (фильтрует входной список). Можно комбинировать Pre+Post на одном методе.

>[!question]- @Secured и @RolesAllowed чем отличаются от @PreAuthorize
>Главное: @PreAuthorize = ВЫРАЖЕНИЕ (SpEL), @Secured/@RolesAllowed = просто СПИСОК РОЛЕЙ («есть одна из», семантика OR).
>
>| | @PreAuthorize | @Secured | @RolesAllowed |
>| откуда | Spring Security | Spring Security (старая) | JSR-250 / Jakarta (стандарт Java) |
>| принимает | SpEL | список строк-authority | список строк-ролей |
>| args / principal / @bean | ✅ | ❌ | ❌ |
>| флаг в @EnableMethodSecurity | prePostEnabled (дефолт) | securedEnabled | jsr250Enabled |
>| префикс ROLE_ | как напишешь (hasRole +, hasAuthority −) | НЕ добавляет (строка как есть) | ДОБАВЛЯЕТ ROLE_ |
>
>@Secured/@RolesAllowed НЕ умеют: смотреть аргументы (#eventId), звать бины (@accessService...), комбинировать and/or, SpEL вообще. Их потолок — проверка роли (как URL-правила). Проверку ВЛАДЕНИЯ соревнованием ими не сделать.
>```java
>@PreAuthorize("hasAuthority('Organizer') and @accessService.doesEventBelongToOrganizer(principal.loginId, #eventId)")
>@Secured("Organizer")        // только: есть ли authority "Organizer"
>@RolesAllowed("Organizer")   // только: есть ли роль → "ROLE_Organizer"!
>```
>⚠️ ROLE_ в этом проекте (authority = "Organizer" БЕЗ префикса):
>- `@PreAuthorize("hasAuthority('Organizer')")` ✅
>- `@Secured("Organizer")` ✅ (строка как есть)
>- `@RolesAllowed("Organizer")` ❌ молчаливый 403 (Jsr250 добавляет ROLE_ → ищет "ROLE_Organizer", те же грабли что hasRole)
>
>Прочее: @RolesAllowed переносим (Java-стандарт), @Secured/@PreAuthorize — чисто Spring. Все три — только «до» (аналога @PostAuthorize нет). @Secured считается устаревшим в пользу @PreAuthorize.
>Когда что: @PreAuthorize — дефолт (любая логика); @Secured/@RolesAllowed — только тривиальное «одна из ролей». В strikerstat (authority без префикса) из простых годится @Secured, НЕ @RolesAllowed.

>[!question]- @PreAuthorize и eventId внутри DTO
>`#eventId` — переменная-АРГУМЕНТ по имени. Если аргумент `AuthorizeEventDto dto`, то `#eventId` не существует → null → проверка провалится (молчаливый 403).
>Достать поле из DTO — навигацией: `#dto.eventId` → `dto.getEventId()`. Глубже: `#dto.fighter.id`.
>| eventId где | SpEL |
>| отдельный аргумент | `#eventId` |
>| поле в DTO | `#dto.eventId` |
>⚠️ Для чтения `#dto.eventId` Spring сначала десериализует тело в объект → JSON парсится ДО проверки прав (в отличие от URL-правил, которые до контроллера).

>[!question]- Две @PutMapping("/update") → приложение не стартует
>Два метода на одном пути → `IllegalStateException: Ambiguous mapping`. Не про Security — про Spring MVC. Лечится разными путями (`/update`, `/update-from-dto`).

>[!question]- @PreAuthorize на КЛАССЕ — можно ли проверять владение во всех методах
>На класс повесить можно — применится ко всем методам. Но две ловушки:
>1. **method-level ПЕРЕОПРЕДЕЛЯЕТ class-level**, а НЕ складывается через and. Метод со своим @PreAuthorize игнорирует классовый → роль придётся повторять в каждом методе.
>2. **Одно выражение на класс не адаптируется к сигнатурам**: `#eventId` для одного метода есть, для метода с DTO — нет (→ null → 403). Class-level годится только для проверок БЕЗ аргументов (роль `hasAuthority('Organizer')`).
>Вывод: роль — на класс ок; владение (зависит от eventId) — только на метод.

>[!question]- Можно ли склеить 2 сигнатуры в одном class-level @PreAuthorize через OR
>Попытка: `... and (@accessService.doesEventBelongToOrganizer(principal.loginId, #dto.eventId) or @accessService.doesEventBelongToOrganizer(principal.loginId, #eventId))`.
>**В таком виде НЕТ — бросит исключение (≈500), а не 403.** Причины по уровням:
>1. **Навигация по свойству на null бросает.** Для метода с `eventId` (без `dto`): `#dto` → null (ок), но `#dto.eventId` → `SpelEvaluationException EL1007E` (property on null). А `or` НЕ спасает: короткое замыкание помогает, только если левый операнд = true; если он **бросает** — исключение летит вверх до `or`. (С `#eventId` такого нет: несуществующая ПЕРЕМЕННАЯ → null без броска; бросает именно `.поле` на null-объекте.)
>2. **Фикс — null-safe навигация `?.`:** `#dto?.eventId`. Тогда выражение перестаёт падать (null вместо броска).
>3. **Но даже с `?.` — плохая идея:**
>   - class-level применяется ко ВСЕМ методам → у метода БЕЗ eventId/dto оба операнда null → `doesEventBelongToOrganizer(loginId, null)` → JPQL `WHERE eventId = null` → всегда false → **403 на методах, которым проверка не нужна**. Аспект умеет «не нашёл eventId → пропустить», OR-выражение — НЕТ (всегда обязано вернуть boolean).
>   - не масштабируется (3-я сигнатура = ещё `or`), двойной запрос в БД, stringly-typed, остаются method-overrides-class и self-invocation.
>Итог: склеить под N сигнатур через `or` + `?.` технически можно, но «пропускать неподходящие методы» @PreAuthorize не умеет — ровно поэтому для «одно правило на класс поверх разнородных методов» берут аспект.

# Выбор механизма авторизации (URL / @PreAuthorize / аспект)

> AOP-механика аспекта (как достаёт eventId через `joinPoint.getArgs()`+рефлексия, pointcut `@within`, advice-типы `@Before`/`@AfterReturning`, self-invocation) вынесена в [[Spring AOP]] → «Авторизация через аспект (механика)». Здесь — только выбор способа и слои авторизации.

>[!question]- Почему @PreAuthorize не заменяет аспект (signature-agnostic поиск)
>@PreAuthorize НЕ умеет «искать eventId в любой сигнатуре», а аспект умеет (см. [[Spring AOP]]):
>1. Class-level @PreAuthorize — ОДНА статическая SpEL-строка на все методы, не подстроится.
>2. SpEL в method-security даёт только ИМЕНОВАННЫЕ параметры (`#eventId`), нет generic-массива аргументов (как `#root.args` в @Cacheable).
>3. Нет рефлексивной эвристики (SpEL ходит по `#dto.eventId`, но не «найди поле eventId где угодно»).
>Суть: аспект = код (адаптируется в рантайме); @PreAuthorize = декларативная строка, привязанная к именам. «Одно правило на класс поверх разнородных сигнатур» — это про аспект.

>[!question]- Два слоя авторизации в проекте (URL-роль + аспект-владение)
>| | filterChain (URL) | AOP-аспект |
>| когда | до контроллера (фильтр) | на входе в метод (прокси) |
>| вопрос | «какая роль?» (вертикальный доступ) | «это твой объект?» (горизонтальный) |
>| видит | URL + роль | аргументы метода + БД |
>URL-правило `hasAuthority('Organizer')` пропустит ЛЮБОГО организатора, но не отличит «своё/чужое соревнование» — для этого нужен eventId из аргументов + запрос в БД. Поэтому появляется 2-й слой.
>В strikerstat method-security сделана НЕ через @PreAuthorize, а самописными аспектами (`@CheckOrganizerAccess` с eventIdParamIndex/eventIdFieldName) — ради единого декларативного чека поверх разнородных сигнатур.

>[!question]- Что выбрать для проверки на каждом методе: явный вызов / @PreAuthorize / аспект
>**Дефолт: гибрид — @PreAuthorize как триггер + логика в бине** (`@PreAuthorize("@accessService.doesEventBelongToOrganizer(principal.loginId, #eventId)")`). Декларативно и заметно в ревью (fail-closed), но логика в обычном бине → тестируема и отлаживаема (а не в строке SpEL).
>- **Явный вызов метода** — для СЛОЖНОЙ логики, плохо ложащейся в SpEL. Минус: легко забыть → тихая дыра (fail-open); компенсировать ArchUnit-тестом.
>- **Свой аспект** — только когда НЕ хочешь трогать каждый метод (одно правило на класс поверх многих сигнатур). Если размечаешь каждый метод — аспект теряет смысл (вся магия без бонуса DRY).
>⚠️ Не держать все 3 механизма параллельно в бою (URL + аспект + @PreAuthorize) — трудно понять «кто же пускает». В strikerstat принята связка URL + аспект.