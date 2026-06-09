# Spring AOP

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