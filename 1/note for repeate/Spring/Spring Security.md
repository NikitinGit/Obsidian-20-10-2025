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