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