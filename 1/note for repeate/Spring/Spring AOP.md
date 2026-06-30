# Spring AOP
1. [ ] вопросы от нейронки 
>[!question]- **AOP** - это
> **парадигма программирования**, которая позволяет выделить сквозную функциональность (cross-cutting concerns) в отдельные модули (аспекты).
> «Сквозная функциональность» (cross‑cutting concern) — это такая часть поведения программы, которая нужна во многих местах и слоях приложения сразу, а не живёт в одном чётко выделенном модуле или классе
> Типичные примеры сквозной функциональности: логирование, аутентификация и проверка прав доступа, обработка исключений, транзакции, кеширование. Их код «размазывается» по контроллерам, сервисам, репозиториям и т.д., из‑за чего он дублируется и переплетается с бизнес‑логикой. АОП как раз позволяет собрать такой повторяющийся «размазанный» код в отдельные аспекты и подключать его декларативно в нужных точках выполнения.[](http://www.finecosoft.ru/spring-aop) 

>[!question]- Что такое `@Aspect` и Spring AOP?
> `@Aspect` — аннотация из AspectJ (`org.aspectj.lang.annotation.Aspect`), которая помечает класс как **аспект** — место со сквозной логикой (cross-cutting concern), выполняющейся "вокруг" обычных методов без загрязнения их кода.
> Сам по себе `@Aspect` ничего не делает — это просто маркер. Чтобы Spring подхватил аспект, нужно:
> 1. `@Component` (или другой стереотип) — чтобы Spring создал бин.
> 2. Включённый `@EnableAspectJAutoProxy` (в Spring Boot включается автоматически при наличии `spring-boot-starter-aop`).

>[!question]- Какие бывают типы advice?
> - `@Before` — выполняется **перед** целевым методом.
> - `@After` — после (всегда, даже при исключении).
> - `@AfterReturning` — после успешного возврата.
> - `@AfterThrowing` — после выброса исключения.
> - `@Around` — оборачивает метод полностью (можно решать, вызывать ли его).

>[!question]- Что такое pointcut и как он связывается с аннотацией?
> Pointcut — это выражение, описывающее, **какие методы перехватывать**. Пример из проекта:
> ```java
> @Before("@annotation(checkOrganizerAccess)")
> public void checkOrganizerAccess(JoinPoint joinPoint, CheckOrganizerAccess checkOrganizerAccess) { ... }
> ```
> - `"@annotation(checkOrganizerAccess)"` — перехватывать любой метод, помеченный аннотацией `CheckOrganizerAccess`.
> - Имя `checkOrganizerAccess` в выражении связывается с параметром метода — Spring передаст в аспект конкретный экземпляр аннотации с её значениями.
> - `JoinPoint` даёт доступ к контексту вызова: аргументам (`joinPoint.getArgs()`), сигнатуре метода и т.д.

>[!question]- Как создаётся своя аннотация для AOP?
> ```java
> @Target(ElementType.METHOD)
> @Retention(RetentionPolicy.RUNTIME)   // ОБЯЗАТЕЛЬНО RUNTIME — иначе аспект её не увидит
> public @interface CheckOrganizerAccess {
>     int eventIdParamIndex() default -1;
>     String eventIdFieldName() default "";
> }
> ```
> Сама аннотация — просто маркер. Логику обработки пишет аспект.

>[!question]- Зачем `@Retention(RetentionPolicy.RUNTIME)` и что будет без него?
> **Нужен. Без него Spring AOP не сработает — молча.**
>
> `@Retention` определяет, до какой стадии жизни доживает аннотация:
> | Значение | Видна где | Кто использует |
> |---|---|---|
> | **`SOURCE`** | только в `.java`, исчезает после компиляции | `@Override`, `@SuppressWarnings`, Lombok |
> | **`CLASS`** (**по умолчанию!**) | в `.class`, но JVM не подгружает в рантайме | байткод-инструменты (ASM, ByteBuddy compile-time) |
> | **`RUNTIME`** | в байткоде + доступна через рефлексию | Spring, JUnit, Jackson |
>
> **Главный подводный камень:** значение **по умолчанию — `CLASS`, а не `RUNTIME`**. Если `@Retention` не указать вообще — аннотация будет в байткоде, но Spring её **не увидит**.
>
> **Почему именно `RUNTIME` нужен Spring AOP:**
> Spring AOP работает через прокси + рефлексию. Внутри он делает примерно:
> ```java
> Annotation ann = clazz.getAnnotation(CheckAuthorization.class);
> ```
> `Class.getAnnotation(...)` возвращает аннотацию **только при `RetentionPolicy.RUNTIME`**. С `SOURCE`/`CLASS` получишь `null`, и аспект не зацепится.
>
> То же касается всех pointcut'ов `@annotation(...)`, `@within(...)`, `@target(...)` — они опираются на рефлексию.
>
> **Что будет, если убрать `@Retention`:**
> - Компиляция пройдёт без ошибок.
> - В байткоде аннотация будет.
> - Но `getAnnotation(...)` вернёт `null`.
> - Аспект **молча не сработает**. Никаких ошибок в логах — самая коварная часть.
>
> **Когда `RUNTIME` НЕ нужен:**
> 1. Lombok-подобные аннотации, обрабатываемые **аннотационным процессором** на этапе компиляции (`@Getter`, `@Builder`) — достаточно `SOURCE`.
> 2. **AspectJ compile-time / load-time weaving** (не Spring AOP!) умеет вшивать аспекты в байткод — ему хватает `CLASS`.
>
> В подавляющем большинстве случаев в Spring-проектах для своих аннотаций нужен `@Retention(RetentionPolicy.RUNTIME)`.

>[!question]- Почему аспект с `@annotation(...)` НЕ ловит аннотацию, повешенную на класс?
> Pointcut **`@annotation(X)`** ищет аннотацию **только на самом методе**. Аннотации на классе он игнорирует.
>
> **Пример проблемы:**
> ```java
> @Target(ElementType.TYPE)           // аннотация только на классы
> public @interface CheckWithoutParam { }
>
> @CheckWithoutParam                   // повешена на КЛАСС
> public class EventController { ... }
>
> @Before("@annotation(checkWithoutParam)")   // ← НЕ сработает
> public void check(JoinPoint jp, CheckWithoutParam checkWithoutParam) { ... }
> ```
> Тишина — потому что `@annotation(...)` ищет метод с этой аннотацией, а её там нет.
>
> **Решение** — использовать `@within(...)` или `@target(...)`:
> ```java
> @Before("@within(checkWithoutParam)")
> public void check(JoinPoint jp, CheckWithoutParam checkWithoutParam) { ... }
> ```
> Теперь аспект сработает **на каждый вызов любого метода** класса, помеченного `@CheckWithoutParam`.

>[!question]- Designator'ы AspectJ для работы с аннотациями — таблица
> | Designator | Что ловит | Когда использовать |
> |---|---|---|
> | **`@annotation(X)`** | методы с аннотацией **`@X` прямо на методе** | аннотация навешана на метод |
> | **`@within(X)`** | методы, **объявленные в классе** с `@X` (статически, по declaring type) | аннотация на классе |
> | **`@target(X)`** | методы, у которых **runtime-тип объекта** имеет `@X` | аннотация на классе, важно для наследования / прокси |
> | **`@args(X)`** | методы, у которых **аргумент** имеет `@X` на своём классе | редко |
> | **`bean(name)`** | методы конкретного Spring-бина по имени | редко |
>
> **Запомнить разницу `@within` vs `@target`:**
> - `@within` — "где **объявлен** метод" (compile-time, declaring class).
> - `@target` — "какого типа **сам объект**" (runtime, реальный класс получателя — учитывает наследование).
>
> В 99% случаев для аннотации на классе достаточно `@within`.
>
> **Комбинирование** — если хочется ловить аннотацию и на классе, и на методе:
> ```java
> @Before("@annotation(ann) || @within(ann)")
> public void check(JoinPoint jp, MyAnnotation ann) { ... }
> ```
> Альтернатива — `AnnotatedElementUtils.findMergedAnnotation(...)` из Spring для поиска по иерархии.

>[!question]- Какие ещё бывают pointcut designator'ы (не про аннотации)?
> - **`execution(...)`** — самый распространённый, по сигнатуре метода:
>   `execution(public * com.example.service.*.*(..))` — все public методы любых классов в `service`.
> - **`within(com.example.service..*)`** — все методы пакета (без аннотации).
> - **`this(Type)`** — proxy-объект имеет указанный тип/интерфейс.
> - **`target(Type)`** — целевой объект (за прокси) имеет тип.
> - **`args(String, ..)`** — методы с такими типами аргументов.
> - Можно комбинировать через `&&`, `||`, `!`.

>[!question]- Можно ли передать параметры в аспект БЕЗ собственной аннотации?
> Да. Аннотация — лишь один из способов выбора методов. Параметры в advice можно получить через `JoinPoint` или через designator'ы `args`/`target`/`this`.
>
> **Пример исходного аспекта без аннотации** (перехватывает все методы контроллера по `execution`):
> ```java
> @Before("execution(* com.example.testlinux.controller.AspectController.*(..))")
> public void check() {
>     eventServiceTest.testAspect();
> }
> ```

>[!question]- Способ 1 — `JoinPoint` (универсальный доступ)
> Самый простой. Не требует менять pointcut. `JoinPoint` — это **первый параметр advice** (если он есть), AspectJ подставляет его автоматически.
> ```java
> @Before("execution(* AspectController.*(..))")
> public void check(JoinPoint joinPoint) {
>     Object[] args = joinPoint.getArgs();                           // все аргументы
>     MethodSignature sig = (MethodSignature) joinPoint.getSignature();
>     String methodName = sig.getName();
>     String[] paramNames = sig.getParameterNames();                 // нужен -parameters
>     Class<?>[] paramTypes = sig.getParameterTypes();
>     Object target = joinPoint.getTarget();                         // целевой бин
>     log.info("intercepted {} args={}", methodName, Arrays.toString(args));
> }
> ```
> **Плюсы:** работает с любой сигнатурой. **Минусы:** `Object[]`, нужен каст.

>[!question]- Способ 2 — `args(...)` (типизированный биндинг)
> Если знаешь сигнатуру и хочешь типизированный параметр:
> ```java
> @Before("execution(* AspectController.*(..)) && args(eventId)")
> public void check(Integer eventId) {
>     log.info("eventId = {}", eventId);
> }
> ```
> Тип `Integer` AspectJ берёт из сигнатуры advice. Pointcut сработает только на методы с одним аргументом `Integer`.
>
> **Wildcards в `args`:**
> - `..` — любое количество остальных аргументов: `args(eventId, ..)`.
> - `*` — любой ОДИН аргумент: `args(eventId, *)`.
>
> **Плюсы:** типизировано. **Минусы:** жёстко привязано к форме сигнатуры — изменишь её, аспект тихо перестанет срабатывать.

>[!question]- Способ 3 — `target(...)` и `this(...)`
> Получить сам бин/прокси.
> ```java
> @Before("execution(* AspectController.*(..)) && target(controller)")
> public void check(AspectController controller) {
>     log.info("intercepted bean {}", controller);
> }
> ```
> - **`target(...)`** — реальный целевой объект (за прокси).
> - **`this(...)`** — сам прокси (то, что в Spring-контейнере).

>[!question]- Способ 4 — комбинация всего
> Можно брать сразу всё. Порядок: сначала `JoinPoint` (опционально), потом параметры из designator'ов **в том же порядке, что в pointcut'е**.
> ```java
> @Before("execution(* AspectController.*(..)) && args(eventId) && target(controller)")
> public void check(JoinPoint jp, Integer eventId, AspectController controller) {
>     log.info("method={}, eventId={}, bean={}",
>         jp.getSignature().getName(), eventId, controller);
> }
> ```

>[!question]- Способ 5 — поиск параметра по имени через рефлексию
> Если методы разной формы, но иногда есть параметр с именем `eventId`:
> ```java
> @Before("execution(* AspectController.*(..))")
> public void check(JoinPoint jp) {
>     MethodSignature sig = (MethodSignature) jp.getSignature();
>     String[] names = sig.getParameterNames();   // требует -parameters
>     Object[] args = jp.getArgs();
>     for (int i = 0; i < names.length; i++) {
>         if ("eventId".equals(names[i]) && args[i] instanceof Integer e) {
>             log.info("eventId = {}", e);
>         }
>     }
> }
> ```

>[!question]- Получить аннотации параметров (`@RequestParam`, `@PathVariable`)
> ```java
> MethodSignature sig = (MethodSignature) jp.getSignature();
> Annotation[][] paramAnnotations = sig.getMethod().getParameterAnnotations();
> // paramAnnotations[i] — массив аннотаций i-го параметра
> ```

>[!question]- Сводная таблица: способы передачи параметров в advice
> | Способ | Где указывается | Типизация | Гибкость |
> |---|---|---|---|
> | `JoinPoint.getArgs()` | параметр advice | `Object[]`, нужен каст | максимальная |
> | `args(name)` | в pointcut + параметр advice | типизированно | средняя |
> | `target(name)` / `this(name)` | в pointcut + параметр advice | да, по типу | для доступа к бину/прокси |
> | `MethodSignature` + рефлексия | внутри advice | через `instanceof` | максимальная, можно по имени параметра |
>
> **Вывод:** аннотация на методе — это лишь способ **выбрать**, какие методы перехватывать (`@annotation(X)`). К **передаче параметров в advice** она прямого отношения не имеет — параметры тащим из `JoinPoint` или биндим через `args`/`target`/`this`.

>[!question]- Как получить eventId без `eventIdParamIndex`?
> Текущая реализация хрупкая — переставил параметры местами и сломалось. Есть лучшие способы.
>
> **1. По имени параметра через `MethodSignature`:**
> ```java
> MethodSignature sig = (MethodSignature) joinPoint.getSignature();
> String[] paramNames = sig.getParameterNames();
> Object[] args = joinPoint.getArgs();
> for (int i = 0; i < paramNames.length; i++) {
>     if ("eventId".equals(paramNames[i]) && args[i] instanceof Integer e) {
>         eventId = e; break;
>     }
> }
> ```
> Требует компиляции с флагом `-parameters` (в Spring Boot включён по умолчанию).
>
> **2. SpEL (как `@PreAuthorize` в Spring Security):**
> ```java
> public @interface CheckOrganizerAccess {
>     String value() default "#eventId";    // SpEL-выражение
> }
> ```
> ```java
> SpelExpressionParser parser = new SpelExpressionParser();
> StandardEvaluationContext ctx = new StandardEvaluationContext();
> for (int i = 0; i < names.length; i++) ctx.setVariable(names[i], args[i]);
> Integer eventId = parser.parseExpression(ann.value()).getValue(ctx, Integer.class);
> ```
> Использование: `@CheckOrganizerAccess("#eventId")`, `@CheckOrganizerAccess("#request.eventId")`, `@CheckOrganizerAccess("#dto.event.id")`.
>
> **3. Аннотация на параметре:**
> ```java
> public ResponseEntity<?> aspect(@EventId @RequestParam Integer eventId) { ... }
> ```
> В аспекте перебирать `signature.getMethod().getParameterAnnotations()` и искать `@EventId`.
>
> **Рекомендация:** SpEL — самый гибкий вариант, покрывает оба сценария (прямой `Integer` и поле DTO) одной аннотацией.

>[!question]- Когда поле DTO работает, а прямой `Integer` — нет?
> Если в аспекте искать `eventId` через рефлексию по имени поля:
> ```java
> Field field = obj.getClass().getDeclaredField("eventId");
> ```
> Для метода `aspect(@RequestParam Integer eventId)` единственный аргумент — `Integer`. У `Integer` **нет поля `eventId`** (только внутреннее `value`) → `NoSuchFieldException` → `eventId = null`.
>
> Этот режим (`eventIdFieldName`) работает только когда в метод передаётся **DTO-объект**, в котором есть поле `eventId`:
> ```java
> public class EventUpdateRequest {
>     private Integer eventId;
>     ...
> }
> @CheckOrganizerAccess(eventIdFieldName = "eventId")
> public ResponseEntity<?> update(@RequestBody EventUpdateRequest request) { ... }
> ```

>[!question]- Как происходит вызов в рантайме?
> 1. Spring видит `@Aspect` бин и при создании целевого бина (например, контроллера) оборачивает его в **прокси** (CGLIB или JDK dynamic proxy).
> 2. При запросе вызывается не сам контроллер, а **прокси**.
> 3. Прокси видит аннотацию на методе → срабатывает pointcut.
> 4. До вызова реального метода выполняется код аспекта.
> 5. Если аспект бросает исключение — реальный метод **не выполнится**.
> 6. Иначе — управление передаётся в целевой метод.

>[!question]- Почему AOP не работает при self-invocation?
> Если из одного метода бина вызвать `this.otherMethod()`, помеченный аннотацией — **прокси обойдётся стороной**, аспект не вызовется.
> Прокси перехватывает только вызовы **извне** (через ссылку на Spring-бин). Внутри объекта `this` указывает на сам объект, а не на прокси-обёртку.
> Решения: вынести метод в другой бин, либо инжектить сам себя через `@Autowired private SelfType self;` и вызывать `self.otherMethod()`.

>[!question]- Почему `@Aspect` работает без `@EnableAspectJAutoProxy` и без явной зависимости `spring-boot-starter-aop` в pom.xml?
> Две причины — транзитивные зависимости + автоконфигурация Spring Boot.
>
> **1. Зависимость подтянулась транзитивно:**
> ```
> spring-boot-starter-data-jpa
>  └─ spring-boot-starter-aop
>      └─ org.aspectj:aspectjweaver
> ```
> Spring Data JPA сам использует AOP (для `@Transactional`, репозиториев), поэтому тянет starter-aop как обязательную зависимость. Проверить можно через `mvn dependency:tree`.
>
> **2. Автоконфигурация `AopAutoConfiguration`:**
> ```java
> @AutoConfiguration
> @ConditionalOnProperty(prefix = "spring.aop", name = "auto", havingValue = "true", matchIfMissing = true)
> public class AopAutoConfiguration {
>     @Configuration(proxyBeanMethods = false)
>     @ConditionalOnClass(Advice.class)        // есть aspectjweaver на classpath
>     @EnableAspectJAutoProxy(proxyTargetClass = true)
>     static class AspectJAutoProxyingConfiguration { ... }
> }
> ```
> - `@ConditionalOnClass(Advice.class)` — конфиг включается, если в classpath есть `aspectjweaver`.
> - `matchIfMissing = true` — по умолчанию AOP включён; отключить можно `spring.aop.auto=false` в `application.properties`.
> - Внутри уже стоит `@EnableAspectJAutoProxy(proxyTargetClass = true)` — Boot сделал это за тебя.
> - `proxyTargetClass = true` означает использование **CGLIB-прокси** (наследованием), а не JDK-прокси через интерфейсы — поэтому контроллеры без интерфейса спокойно проксируются.
>
> **Лучшая практика:** не полагаться на транзитивщину — добавить starter-aop явно:
> ```xml
> <dependency>
>     <groupId>org.springframework.boot</groupId>
>     <artifactId>spring-boot-starter-aop</artifactId>
> </dependency>
> ```
> Если когда-нибудь уберёшь JPA — AOP не сломается молча.

>[!question]- Альтернатива AOP для гейта по URL — `HandlerInterceptor`
> Когда нужно отрезать группу HTTP-ручек по простому условию (фича-флаг, право доступа на префикс, авторизация по заголовку), часто лучше **`HandlerInterceptor`**, а не аспект. Это нативный механизм Spring MVC, без прокси и pointcut'ов.
>
> **Реализация — `preHandle()` возвращает `false`, обработка дальше не идёт:**
> ```java
> @Component
> @RequiredArgsConstructor
> public class ChallengesFeatureFlagInterceptor implements HandlerInterceptor {
>
>     private final ChallengeFeatureFlagService challengeFeatureFlagService;
>
>     @Override
>     public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
>         if (challengeFeatureFlagService.isEnabled()) {
>             return true;
>         }
>         response.setStatus(HttpStatus.NOT_FOUND.value());
>         return false;   // контроллер вызван НЕ будет
>     }
> }
> ```
>
> **Регистрация — в `WebMvcConfigurer`** с path-pattern и исключениями:
> ```java
> @Configuration
> public class WebConfig implements WebMvcConfigurer {
>
>     @Autowired private ChallengesFeatureFlagInterceptor challengesFeatureFlagInterceptor;
>
>     @Override
>     public void addInterceptors(InterceptorRegistry registry) {
>         registry.addInterceptor(challengesFeatureFlagInterceptor)
>                 .addPathPatterns("/api/fighter/challenges/**") // interceptor применится к любому HTTP-запросу, чей URL совпадает с паттерном по AntMatcher. Совершенно не важно, в каком контроллере объявлен метод.
>                 .excludePathPatterns("/api/fighter/challenges/feature-state");// исключение - этот ендпоинт не будет перехватываться
>     }
> }
> ```
>
> **Три метода жизненного цикла:**
> | Метод | Когда вызывается | Зачем |
> |---|---|---|
> | `preHandle()` | до контроллера | гейт/авторизация; `return false` останавливает цепочку |
> | `postHandle()` | после контроллера, до рендера view | дополнить `ModelAndView` (для MVC-views, не для REST) |
> | `afterCompletion()` | после полного завершения запроса (включая исключения) | освободить ресурсы, метрики, логирование |
>
> **Когда interceptor лучше, чем `@Aspect`:**
> - Условие — про **URL / HTTP**, а не про метод бизнес-логики.
> - Нужен **path-pattern** (`/api/foo/**`) — это родная фича `InterceptorRegistry`, в AOP пришлось бы матчить через имена методов.
> - Хочется чисто декларативно исключить отдельные пути (`.excludePathPatterns(...)`).
> - Не хочется проксировать контроллер (CGLIB-прокси добавляет накладные).
>
> **Когда `@Aspect` лучше:**
> - Условие — про **бизнес-метод**, а не про URL (например, гейт нужен и при внутреннем вызове сервиса из другого сервиса).
> - Нужно перехватывать не только контроллеры, а любые бины (`@Service`, `@Component`).
> - Хочется декларативный маркер аннотацией (`@RequiresFeature("xxx")`).
>
> **Что куда не дотягивается:**
> - **`HandlerInterceptor`** срабатывает **только на HTTP-вход**. Если другой Spring-бин позовёт `service.doStuff()` напрямую — гейт не сработает. Это и плюс (контроллер чистый), и минус (нет защиты от внутренних вызовов).
> - **`@Aspect`** не работает при **self-invocation** (`this.method()` обходит прокси — см. вопрос выше). Interceptor этим не страдает — он срабатывает на сам HTTP-запрос, а не на вызов метода.
>
> **Гибрид (если хочется и то и другое):**
> 1. `HandlerInterceptor` — гейт на уровне HTTP (защита API).
> 2. Дополнительная проверка в сервисе первой строкой публичного метода (как `requireMessagingFeatureAccess(...)` в `PrivateMessagingService`) — защита от внутренних вызовов в обход контроллера.

---

## Spring Boot и стартовые зависимости

Spring Boot работает через механизм **автоконфигурации** (`@EnableAutoConfiguration`, который входит в `@SpringBootApplication`). При старте Spring сканирует classpath и активирует `*AutoConfiguration` классы по условиям `@ConditionalOnClass`, `@ConditionalOnMissingBean`, `@ConditionalOnProperty` и т.д.

**Starter-зависимости** — это "наборы" артефактов, объединённые под одним именем. Они тянут согласованные версии библиотек + соответствующие AutoConfiguration.

>[!question]- Какие основные starter-зависимости тянет Spring Boot?
> | Стартер | Что подключает |
> |---|---|
> | `spring-boot-starter` | Базовое ядро: `spring-core`, `spring-context`, logging (Logback), YAML (SnakeYAML), `spring-boot-autoconfigure` |
> | `spring-boot-starter-web` | Spring MVC, встроенный Tomcat, Jackson (JSON), валидация (`hibernate-validator`) |
> | `spring-boot-starter-data-jpa` | Hibernate, Spring Data JPA, JDBC, **транзитивно `spring-boot-starter-aop`** |
> | `spring-boot-starter-aop` | `spring-aop`, `aspectjweaver` + автоконфиг `AopAutoConfiguration` |
> | `spring-boot-starter-security` | Spring Security + автоконфиг (`SecurityAutoConfiguration`) |
> | `spring-boot-starter-test` | JUnit 5, Mockito, AssertJ, Spring Test, JsonPath |
> | `spring-boot-starter-validation` | `hibernate-validator` (JSR-380) |
> | `spring-boot-starter-actuator` | Endpoints для мониторинга (`/health`, `/metrics`, `/info`) |
> | `spring-boot-starter-data-redis` | Lettuce / Jedis + Spring Data Redis |
> | `spring-boot-starter-cache` | Абстракция кеширования + автоконфиг (`CacheAutoConfiguration`) |
> | `spring-boot-starter-thymeleaf` | Thymeleaf шаблонизатор |
> | `spring-boot-starter-mail` | JavaMail для отправки писем |
> | `spring-boot-starter-quartz` | Quartz Scheduler |
> | `spring-boot-starter-batch` | Spring Batch для пакетной обработки |
> | `spring-boot-starter-webflux` | Reactive стек (Netty + Project Reactor), альтернатива MVC |

>[!question]- Что важно про управление версиями?
> `spring-boot-starter-parent` (или BOM `spring-boot-dependencies`) задаёт **согласованные версии** всех Spring и сторонних библиотек. Поэтому в `pom.xml` дочерних зависимостей версию указывать **не нужно** — её определяет parent.
> Свойства можно переопределять через `<properties>`: например, `<jackson.version>2.17.0</jackson.version>`.

>[!question]- Как узнать, какие AutoConfiguration сработали?
> 1. Запустить приложение с флагом `--debug` (или `debug=true` в `application.properties`) — в логе появится **CONDITIONS EVALUATION REPORT** с разделами:
>     - `Positive matches` — какие автоконфиги активировались.
>     - `Negative matches` — какие отвалились и почему.
> 2. Через actuator endpoint `/actuator/conditions` (если подключён `starter-actuator`).
> 3. Команда `mvn dependency:tree` показывает транзитивные зависимости — что вообще оказалось в classpath.

---

## Получить данные в аспекте БЕЗ объявления параметра в методе контроллера

>[!question]- Как избежать "мёртвых параметров" в сигнатуре метода?
> **Проблема:** если `eventId` нужен только аспекту (для авторизации), а внутри метода контроллера не используется — приходится писать `Integer eventId` в сигнатуре только ради аспекта. Параметр объявлен, но не используется. Это code smell.
>
> Решений несколько — выбирать по контексту.

>[!question]- Способ 1 — читать `HttpServletRequest` прямо в аспекте через `RequestContextHolder`
> Аспект сам залезает в текущий HTTP-запрос и достаёт параметр. Контроллер вообще не знает про `eventId`.
> ```java
> import org.springframework.web.context.request.RequestContextHolder;
> import org.springframework.web.context.request.ServletRequestAttributes;
> import org.springframework.web.servlet.HandlerMapping;
>
> @Before("execution(* com.example.testlinux.controller.AspectController.*(..))")
> public void check() {
>     HttpServletRequest request = ((ServletRequestAttributes)
>             RequestContextHolder.currentRequestAttributes()).getRequest();
>     Integer eventId = extractEventId(request);
>     if (eventId != null) eventServiceTest.testAspect(eventId);
> }
>
> private Integer extractEventId(HttpServletRequest request) {
>     // 1. Query string: ?eventId=...
>     String qp = request.getParameter("eventId");
>     if (qp != null) return Integer.valueOf(qp);
>     // 2. Path variable: /api/.../{eventId}/...
>     Map<String, String> uriVars = (Map<String, String>)
>             request.getAttribute(HandlerMapping.URI_TEMPLATE_VARIABLES_ATTRIBUTE);
>     if (uriVars != null && uriVars.containsKey("eventId"))
>         return Integer.valueOf(uriVars.get("eventId"));
>     return null;
> }
> ```
> Контроллер чистый:
> ```java
> @GetMapping("/3")
> public ResponseEntity<String> endPointN3() {     // никакого eventId в сигнатуре!
>     return ResponseEntity.ok("done");
> }
> ```
> **Плюсы:** контроллер чист.
> **Минусы:** "магия" — глядя на метод не видно, что что-то проверяется. Аспект работает только в HTTP-контексте.

>[!question]- Подводный камень `RequestContextHolder` — `IllegalStateException`
> `RequestContextHolder.currentRequestAttributes()` бросит **`IllegalStateException`**, если аспект сработал **вне HTTP-запроса** (например, в @Scheduled, Kafka-listener'е, тесте). Защита:
> ```java
> RequestAttributes attrs = RequestContextHolder.getRequestAttributes();
> if (!(attrs instanceof ServletRequestAttributes)) return;   // не HTTP
> HttpServletRequest request = ((ServletRequestAttributes) attrs).getRequest();
> ```
> При узком pointcut'е `execution(* ...controller.*)` проблема не возникает — контроллеры всегда в контексте запроса.

>[!question]- Способ 2 — `HandlerInterceptor` вместо AOP
> Идиоматичный способ "сделать что-то перед вызовом контроллера на основе HTTP-запроса". Это часть Spring MVC, специально для этого созданная.
> ```java
> @Component
> @RequiredArgsConstructor
> public class OrganizerAccessInterceptor implements HandlerInterceptor {
>     private final AccessService accessService;
>
>     @Override
>     public boolean preHandle(HttpServletRequest request,
>                              HttpServletResponse response,
>                              Object handler) {
>         Integer eventId = extractEventId(request);
>         if (eventId == null) return true;
>
>         Integer organizerLoginId = 4;  // из SecurityContext
>         if (!accessService.doesEventBelongToOrganizer(organizerLoginId, eventId)) {
>             response.setStatus(HttpStatus.FORBIDDEN.value());
>             return false;              // прерывает запрос, контроллер не вызывается
>         }
>         return true;
>     }
> }
> ```
> Регистрация:
> ```java
> @Configuration
> @RequiredArgsConstructor
> public class WebConfig implements WebMvcConfigurer {
>     private final OrganizerAccessInterceptor interceptor;
>
>     @Override
>     public void addInterceptors(InterceptorRegistry registry) {
>         registry.addInterceptor(interceptor).addPathPatterns("/api/test/**");
>     }
> }
> ```
> **Плюсы:** семантически правильно, не нужен AOP, декларативное подключение через `addPathPatterns`/`excludePathPatterns`.
> **Минусы:** работает только на уровне HTTP, не на произвольных методах.

>[!question]- Способ 3 — Spring Security `@PreAuthorize` с SpEL (production)
> Для реальной авторизации в проде:
> ```java
> @PreAuthorize("@accessService.doesEventBelongToOrganizer(principal.id, #eventId)")
> @GetMapping("/3/{eventId}")
> public ResponseEntity<String> endPointN3(@PathVariable Integer eventId) { ... }
> ```
> Можно вообще без параметра — через кастомный `PermissionEvaluator`, который сам читает `HttpServletRequest`.

>[!question]- Сравнение способов получить данные без параметра в методе
> | Способ | Где живёт код | Чистота контроллера | Идиоматичность | Когда выбирать |
> |---|---|---|---|---|
> | Параметр в сигнатуре | в методе | низкая | средняя | когда параметр всё равно нужен |
> | Аспект + `RequestContextHolder` | аспект | высокая | средняя | смешанная логика (HTTP + произвольные методы) |
> | `HandlerInterceptor` | конфиг + перехватчик | максимальная | **высокая** | **только HTTP**, авторизация на эндпоинте |
> | Аннотация + аспект + request | контроллер + аспект | высокая | высокая | хочется явный маркер на методе |
> | Spring Security `@PreAuthorize` | контроллер + SpEL | высокая | **самая высокая** | реальная авторизация в проде |

---

## Над чем вешать аспекты — над контроллером или над сервисом?

>[!question]- Общее правило: на каком слое вешать аннотации/аспекты?
> Зависит от **природы cross-cutting concern**.
>
> **На контроллере** (или через `HandlerInterceptor`):
> - Авторизация на уровне HTTP-эндпоинта (`@PreAuthorize`, кастомные `@CheckAccess`).
> - Rate limiting.
> - Логирование HTTP-запросов/ответов.
> - Валидация входа.
> - Сбор HTTP-метрик (хотя их обычно собирают через actuator/observability).
> - Любая "fail fast" логика — отсечь невалидный запрос до того, как сервис начнёт работу.
>
> **На сервисе** (бизнес-слой):
> - **`@Transactional`** — границы транзакции совпадают с бизнес-операцией, а не с HTTP-запросом.
> - **`@Cacheable` / `@CacheEvict`** — кешируется бизнес-результат, ключ из бизнес-параметров.
> - **`@Retryable`** — повторяем бизнес-операцию (вызов внешнего API), не HTTP-запрос.
> - **`@Async`** — асинхронное выполнение бизнес-метода.
> - **Аудит** — пишем "что произошло в системе", не "кто куда сделал HTTP".
> - **Feature flags** — гейтинг бизнес-логики.
> - **Метрики бизнес-операций** (счётчики, гистограммы).
>
> **Принципы выбора:**
> 1. **Бизнес-инвариант** (транзакция, кеш, retry, аудит бизнес-события) → **сервис**.
> 2. **HTTP-инвариант** (auth на эндпоинте, rate limit, request logging) → **контроллер** или `HandlerInterceptor`.
> 3. **Доступность через разные entry points**: если бизнес-метод может вызываться не только из контроллера (REST + gRPC + scheduler + Kafka listener), аспект должен быть **на сервисе** — иначе через другие пути логика не сработает.
> 4. **Fail fast**: если аспект нужен для отказа в обработке, лучше **выше по стеку** (контроллер/interceptor) — не тратить ресурсы на запуск сервиса.

>[!question]- Конкретные примеры — где обычно
> | Аннотация | Где обычно | Почему |
> |---|---|---|
> | `@Transactional` | **сервис** | транзакция = бизнес-операция, объединяет несколько вызовов репозитория |
> | `@Cacheable`, `@CacheEvict` | **сервис** | кешируется бизнес-результат, ключ — бизнес-параметры |
> | `@Retryable` | **сервис** | повторяем бизнес-вызов внешнего API |
> | `@Async` | **сервис** | бизнес-метод выполняется в фоне |
> | `@PreAuthorize` | **контроллер** (или сервис) | HTTP-слой авторизации; на сервисе — если та же бизнес-логика вызывается из разных entry points |
> | `@CheckOrganizerAccess` (кастомный) | **контроллер** | привязано к HTTP-запросу (URL содержит `eventId`) |
> | `@Audited` | **сервис** | аудируется бизнес-событие |
> | Request logging | **HandlerInterceptor** / **Filter** | чистый HTTP-уровень, не AOP |
> | Rate limiting | **контроллер** / **Filter** | работает на уровне HTTP |
> | Метрики бизнес-операций | **сервис** | измеряем бизнес-действие, не HTTP |

>[!question]- Важный нюанс — self-invocation влияет на выбор слоя
> Аспект **не сработает** при вызове `this.method()` внутри одного бина. Это надо учитывать при выборе слоя:
>
> - Если на **контроллере** висит `@Transactional` — это сразу подозрительно. Контроллер обычно вызывает методы сервиса (через прокси → транзакция работает). Но если внутри контроллера есть несколько шагов с DB, и один зовёт другой через `this.x()` — транзакция не откроется.
> - Если на **сервисе** есть метод `public void a() { b(); }` и `b()` помечен `@Transactional` — **транзакция не откроется** (self-invocation). Решение: вынести `b()` в другой бин или инжектить `self` в сервис.
>
> Поэтому **`@Transactional` лучше вешать на публичные методы сервиса, которые вызываются извне** (из контроллера или другого сервиса), а не на внутренние помощники.

>[!question]- Правильный layered подход (рекомендация)
> ```
> HTTP-запрос
>   │
>   ▼
> ┌─────────────────────────────────────────────┐
> │ Filter / HandlerInterceptor                 │  ← rate limit, request logging,
> │                                             │    CORS, auth token parsing
> └─────────────────────────────────────────────┘
>   │
>   ▼
> ┌─────────────────────────────────────────────┐
> │ Controller                                  │  ← @PreAuthorize (HTTP-уровень),
> │   - валидация входа (@Valid)                │    кастомные @CheckXxx для прав
> │   - маппинг URL → метод                     │
> └─────────────────────────────────────────────┘
>   │
>   ▼
> ┌─────────────────────────────────────────────┐
> │ Service                                     │  ← @Transactional, @Cacheable,
> │   - бизнес-логика                           │    @Retryable, @Async, @Audited,
> │   - оркестрация вызовов                     │    бизнес-метрики
> └─────────────────────────────────────────────┘
>   │
>   ▼
> ┌─────────────────────────────────────────────┐
> │ Repository / External clients               │  ← обычно без аспектов
> └─────────────────────────────────────────────┘
> ```

---

# Авторизация через аспект (механика, на примере EventOwnershipAspect)

> Связано: выбор «аспект vs @PreAuthorize vs явный вызов» как СПОСОБ авторизации — в [[Spring Security]]. Здесь — только AOP-механика «как аспект достаёт данные и когда срабатывает».

>[!question]- Умный аспект: достать eventId независимо от сигнатуры + повесить на КЛАСС
>Аспект — **императивный код** с доступом к `joinPoint.getArgs()` + рефлексия, поэтому может найти eventId в любой сигнатуре (чего НЕ умеет статический @PreAuthorize с `#eventId`/`#dto.eventId`):
>```java
>@Before("@within(checkEventOwnership)")   // pointcut по КЛАССОВОЙ аннотации → все методы класса
>public void check(JoinPoint jp, CheckEventOwnership ann) {
>    // 1) прямой аргумент по имени "eventId":  paramNames[i]=="eventId" && args[i] instanceof Integer
>    // 2) поле "eventId" в объекте-аргументе (DTO): рефлексия getDeclaredField (рекурсивно по иерархии)
>    // eventId не найден → пропуск (метод не про соревнование); чужое → 403; нет auth → 401
>}
>```
>- `@within(ann)` — pointcut: матчит все методы класса, помеченного `@CheckEventOwnership` (TYPE-аннотация). Так одна аннотация на классе покрывает все методы.
>- `joinPoint.getArgs()` + `MethodSignature.getParameterNames()` — доступ ко всем аргументам и их именам.
>- Текущий пользователь: `SecurityContextHolder.getContext().getAuthentication().getPrincipal()` → `Auth` → loginId.
>- Политика «не нашёл eventId → пропустить» нужна, чтобы методы без eventId (листинги, health) не падали.
>Цена: «магия» (поведение спрятано), хрупкая эвристика (опирается на имя `eventId`), подводные камни прокси (self-invocation).

>[!question]- Сделать проверку ПОСЛЕ метода: @Before → @AfterReturning (аналог @PostAuthorize)
>Сменить тип advice:
>
>| advice | когда | видит результат |
>|---|---|---|
>| @Before (сейчас) | до метода | ❌ |
>| @AfterReturning | после успешного возврата | ✅ через `returning` |
>| @AfterThrowing | после исключения | видит исключение |
>| @After | после в любом случае (finally) | ❌ |
>| @Around | оборачивает (до+после, управляет вызовом) | ✅ |
>```java
>@AfterReturning(pointcut = "@within(checkEventOwnership)", returning = "result")
>public void check(JoinPoint jp, CheckEventOwnership ann, Object result) {
>    // теперь eventId можно искать в result (то, что метод ВЕРНУЛ) — аналог returnObject в @PostAuthorize
>}
>```
>Смысл «after» — проверять по тому, что метод ВЕРНУЛ (владелец виден только в загруженном объекте). ⚠️ Тело метода уже выполнилось → побочные эффекты случились (@Transactional откатит БД при броске, но письмо/HTTP — нет) → только для читающих методов.
>⚠️ Если метод возвращает `ResponseEntity<Void>` — в result нет eventId, «after» проверять нечего; нужен метод, возвращающий объект с eventId.

>[!question]- Почему @PreAuthorize не умеет «искать eventId где угодно» (в отличие от аспекта)
>1. Class-level @PreAuthorize — ОДНА статическая SpEL-строка на все методы, не подстроится под разные сигнатуры.
>2. В method-security SpEL доступны только ИМЕНОВАННЫЕ параметры (`#eventId`), нет generic-массива всех аргументов (как `#root.args` в @Cacheable) → нечем «пробежать по аргументам».
>3. Нет рефлексивной эвристики — SpEL ходит по известному пути (`#dto.eventId`), но не «найди поле eventId в любом аргументе».
>Суть: аспект = код (видит всё через getArgs, адаптируется в рантайме); @PreAuthorize = декларативная строка, привязанная к именам. Поэтому «одно правило на класс поверх разнородных сигнатур» — это про аспект.
> Грубо: чем **выше** в стеке — тем больше про HTTP/безопасность/входной фильтр. Чем **глубже** — тем больше про бизнес-инварианты (транзакции, кеш, retry).