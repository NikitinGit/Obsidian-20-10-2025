## Создание и жизненный цикл бинов

>[!question]- `CGLIB` это
>Code Generation Library
> билиотека динамического создания классов и проксиобъектов в рантайме 
> - подменять вызовы методов
> - создавать прокси для @Configuration классов
> - делать AOP (аспектно-ориентированное программирование)
> - внедрять транзакции (@Transactional)
> - реализовывать lazy loading
> - создавать бины без интерфейсов
> Детальный разбор JDK vs CGLIB прокси — [[Proxy Object]]

>[!question]- `@Bean` это
> своего рода фабричный метод - ставится только над методами 
> без @Configuration над классом в котором находится этот бин Методы @Bean НЕ ПРОКСИРУЮТСЯ через CGLIB -  без @Configuration   возможна ошибка нарушения синглтон типа SELF-INVOKED CALL  - внедрение помеченной @Bean зависимости в конструктор 
	 
>[!question]- Фабрика бина (factoryBeanName / factoryMethodName)
> Когда бин создаётся не через `new SomeClass()`, а через вызов метода на другом объекте — в `BeanDefinition` вместо `beanClassName` хранится **ссылка на фабрику**: `factoryBeanName` (кто создаёт) + `factoryMethodName` (какой метод вызвать). Два основных случая:
>
> **1. Обычный `@Bean`-метод** — фабрика это сам `@Configuration`-бин.
> ```java
> @Configuration
> public class GreetingConfig {
>     @Bean
>     public Greeting greetingFromBeanMethod() {
>         return new Greeting("Привет из @Bean-метода");
>     }
> }
> ```
> Реальные поля `BeanDefinition` бина `greetingFromBeanMethod` (проверено запуском, `beanFactory.getBeanDefinition("greetingFromBeanMethod")`):
> ```
> beanClassName: null
> factoryBeanName: greetingConfig
> factoryMethodName: greetingFromBeanMethod
> ```
> `beanClassName` — `null`, потому что класс `Greeting` не инстанцируется напрямую контейнером — Spring вызывает `greetingConfig.greetingFromBeanMethod()`. `factoryBeanName` = имя класса `GreetingConfig` с маленькой буквы (дефолтное имя бина через `Introspector.decapitalize`) — сам `@Configuration`-класс тоже обычный бин в контексте.
>
> **2. `FactoryBean<T>`** — фабрика это не метод, а отдельный интерфейс со своим `getObject()`. Так создаются, например, репозитории Spring Data: `JpaRepositoryFactoryBean.getObject()` строит JDK Dynamic Proxy для интерфейса `MyRepository extends JpaRepository` — у самого интерфейса нет ни класса, ни конструктора, поэтому единственный способ его создать — через фабрику.
>
> Зачем это знать: если в проекте видите бин без конструктора в логах/стектрейсе (как `DemoRepository`) — это верный признак, что он создан через фабрику (метод или `FactoryBean`), а не напрямую через `new`.

>[!question]- `BeanDefinition` ("рецепт") — что реально внутри
> Результат сканирования — не сам бин, а инструкция по его созданию. Важно: это **не** карта "где по проекту этот бин используется" — обратной карты использований Spring нигде не хранит, она эмерджентно возникает во время создания через рекурсивный `getBean()`.
>
> Реальные поля:
> - **`beanClassName`** — какой класс создавать (`null`, если бин через фабрику — см. выше).
> - **scope** — `singleton`/`prototype`/...
> - **`lazyInit`** (boolean) — ленивая инициализация.
> - **`initMethodName`/`destroyMethodName`** — это **НЕ** `@PostConstruct`/`@PreDestroy`! Сюда попадают только явно указанные `@Bean(initMethod=..., destroyMethod=...)` (или XML `init-method`). `@PostConstruct`/`@PreDestroy` в `BeanDefinition` вообще не хранятся — их находит и вызывает отдельный механизм, `CommonAnnotationBeanPostProcessor`, через рефлексию класса уже во время создания бина.
> - **`dependsOn`** — какие бины должны быть созданы раньше.
> - **`factoryBeanName`/`factoryMethodName`** — см. отдельную заметку выше про фабрику бина.
> - **`AnnotationMetadata`** (только у `AnnotatedBeanDefinition`, т.е. у отсканированных `@Component`/`@Service`/`@Repository`) — полная информация обо всех аннотациях класса. У `@Bean`-бинов такого нет — там вместо этого фабрика.
> - **`ConstructorArgumentValues`/`PropertyValues`** — значения конструктора/свойств, заданные **извне** (XML `<constructor-arg>`, либо программно через `BeanDefinitionBuilder.addConstructorArgValue(...)`). Обычная инициализация полей в Java-коде (`private int age = 5;`, присвоение в теле конструктора) сюда НЕ попадает — это просто байткод конструктора, который выполняется на шаге 3, а не хранится в рецепте. Для типичного `@Component` с `@Autowired` эти поля в `BeanDefinition`, как правило, вообще пустые — резолвинг зависимостей происходит динамически при создании бина, а не через заранее сохранённые значения.
>
> Проверено на реальном запуске (`BeanDefinitionRegistryPostProcessor` + `addConstructorArgValue`): сразу после старта `getGenericArgumentValues()` пуст — Spring переносит значение в `getIndexedArgumentValues()` после того, как сопоставил его с конкретной позицией параметра конструктора.

>[!question]-  Порядок инициализации бина 
> Чиcтые бины инициализируются позже тех, в которые они внедренны Сначала создаются бины, от которых зависят другие 
> 1. Обнаружение компонента - `@ComponentScan`,  читает файлы `.class` на жестком диске (Сканирование classpath). 
> Содается список-инструкция BeanDefinition , пока все бины не пройдут этот этап, следующий не начнутся.
> Логировать можно через
>```
>application.properties /  logging.level.org.springframework.context=DEBUG
>```
>Пример вывода 
>```
>Identified candidate component class: file [/home/igor/IdeaProjects/sokcets/SpringBeans/target/classes/org/example/springbeans/bean/life/circle/BusinessService.class]
>//О, я нашел класс! Запишу его в список кандидатов на создание
>``` 
>Сам бин ещё не создан, это только "рецепт" — инструкция по его созданию (а не карта мест использования бина в проекте). `BeanDefinition` — низкоуровневый механизм, с которым бизнес-разработчики (банки, стартапы, не пишущие свои библиотеки/фреймворки для Spring) обычно не работают напрямую — для бизнес-логики, как правило, достаточно конструктора, `@PostConstruct`/`@PreDestroy`, и изредка `@Bean(initMethod=...)` при подключении чужих (сторонних) классов как бинов.
> 2. Вызов Конструктора (`instance = new MyClass()`) (от сюда начинается Построение графа зависимостей - берет список инструкций из Шага 1 и  получает  информачию о том, какие бины  инциализировать первыми а какие потом.  
> ```
> граф зависимостей начинается с вызова конструктора бина А -> если в нем есть иньекция зависимости Б -> вызов конструтора         
  зависимости Б (если в Б конструторе иньексия В зависимости - вызов конструтора В и т.д.) ,после (если есть) вызов @Autowired полей, после (если есть) вызов сеттеров методов иньекций. Б создаётся полностью, весь цикл (конструктор → DI → Aware → 
  postProcessBeforeInitialization/@PostConstruct → afterPropertiesSet → postProcessAfterInitialization) и только после этого начинается создаваться следующая зависимость в А бине. Если бин А инициализирован, но есть бин Х, в котором есть зависимость А - то А не инициализируется и просто подставляется в зависиомть бина Х из кеша .
  >Завершается посторение после populateBean — заполнение @Autowired-полей/сеттеров (внутри — разрешение ЭТИХ зависимостей) , перед postProcessBeforeInitialization (когда все нужные этому бину зависимости точно на месте)
> ```
> )
>    Пример системного лога 
>    ```
> Creating shared instance of singleton bean 'infrastructureBean'
>    ```
> 3. BeanNameAware.setBeanName - бину сообщается его уже назначенное имя
> 4. BeanPostProcessor.postProcessBeforeInitialization - создается отдельный класс реализующий интерфейс BeanPostProcessor 
> 5. Вызов @PostConstruct - Spring ГАРАНТИРОВАННО внедрил все зависимости ДАННОГО БИНА. (База данных / Сеть готовы) - это этап initialization callback — вызывается после конструктора и после внедрения всех зависимостей (DI) данного бина, но до того как бин станет доступен для  использования. 
> 6. InitializingBean.afterPropertiesSet  - «родной» интерфейс самого Spring Framework, который существовал в нем **с самой первой версии (2003 год)**. Чтобы им воспользоваться, ваш класс _обязан_ наследоваться от интерфейса Spring (`implements InitializingBean`).
> 7. BeanPostProcessor.postProcessAfterInitialization - в том же классе где BeanPostProcessor.postProcessBeforeInitialization 
> 8. После того, как все бины прошли эти 6 шагов - срабатывает **`SmartInitializingSingleton.afterSingletonsInstantiated()`** 

>[!question]- как указать какой бин от какого зависит явно если нет обычного механизма DI 
> @DependsOn("infrastructureBean") над классом 

>[!question]- отличие @PostConstruct от afterPropertiesSet
>afterPropertiesSet старше (родной механизм Spring с самого начала), @PostConstruct — новее (пришёл как стандарт Java/JSR-250 и стал
  предпочтительным).
  @PostConstruct/@PreDestroy в Spring появились в Spring Framework 2.5 (2007 год) — именно в этом релизе добавили обширную поддержку аннотаций (@Autowired, @Qualifier,      
  @PostConstruct, @PreDestroy, @Resource, сканирование @Component) через CommonAnnotationBeanPostProcessor.

>[!question]- отличие @PostConstruct от afterSingletonsInstantiated
>afterSingletonsInstantiated срабатывает когда все синглтон бины инициализировались

>[!question]- Aware (Осведомленный, знающий)— что это (шаг между DI и `postProcessBeforeInitialization`)
> Группа маркерных интерфейсов вида `XxxAware`, через которые бин получает доступ не к обычным бизнес-зависимостям (как через `@Autowired`), а к **внутренним объектам самого контейнера**.
>
> Самые частые:
> - **`BeanNameAware`** — `setBeanName(String name)` — сообщает бину его собственное имя в контейнере.
> - **`BeanFactoryAware`** — ссылка на сам `BeanFactory`, в котором бин зарегистрирован.
> - **`ApplicationContextAware`** — ссылка на весь `ApplicationContext` (например, чтобы вручную вызвать `getBean()` изнутри бина).
> - **`EnvironmentAware`** — доступ к `Environment` (профили, свойства `application.yaml`).
> - **`ApplicationEventPublisherAware`** — публикация событий контейнера.
> - **`ResourceLoaderAware`**, **`BeanClassLoaderAware`**, **`MessageSourceAware`** — аналогично, доступ к соответствующей инфраструктуре.
>
> Почему не через `@Autowired`: это не бины бизнес-логики, а сама "начинка" контейнера, поэтому механизм проще — Spring просто проверяет `if (bean instanceof BeanNameAware) { ((BeanNameAware) bean).setBeanName(...); }` и вызывает нужный сеттер напрямую.
>
> Тонкость по порядку: `BeanNameAware`/`BeanClassLoaderAware`/`BeanFactoryAware` вызываются контейнером **напрямую**, до вызова любого `BeanPostProcessor` — отдельный, самый первый под-шаг. А вот `ApplicationContextAware`/`EnvironmentAware`/остальные реализованы через специальный `BeanPostProcessor` (`ApplicationContextAwareProcessor`), зарегистрированный самым первым в очереди — формально они срабатывают уже **внутри** `postProcessBeforeInitialization`, а не строго до него.

>[!question]- Bean "Отдан наружу" - это 
>возвращён из getBean() кому-либо: другому бину, который его затем сохранит в своё поле, или вашему собственному коду в main(),
>который вызовет на  нём метод. Например 
>```
>BusinessService service = context.getBean(BusinessService.class);
>```

>[!question]- Жизненный цикл бина на практике
> Для каждого этапа — что писать и зачем.
>
> **1. Конструктор**
> Только присваивание полей (constructor injection) и лёгкая валидация аргументов. Никаких обращений к другим бинам, сети, БД — граф ещё не достроен, а сам объект в этот момент ещё "сырой" (не прошёл DI/AOP-обёртку).
> ```java
> public BusinessService(InfrastructureBean infra) {
>     this.infra = Objects.requireNonNull(infra);
> }
> ```
> Зачем: `final`-поля → неизменяемость, невозможно создать бин в невалидном состоянии, тесты пишутся как `new BusinessService(mockInfra)` без поднятия Spring-контекста.
>
> **2. `@PostConstruct`**
> Тяжёлая инициализация, которой нужны уже внедрённые зависимости: открыть соединение, прогреть кэш, провалидировать конфиг (`@Value`), подписаться на события.
> ```java
> @PostConstruct
> private void init() {
>     if (apiUrl == null) throw new IllegalStateException("apiUrl не задан");
>     this.httpClient = HttpClient.newHttpClient();
> }
> ```
> Зачем: единственная точка, где гарантированно и зависимости внедрены, и бин ещё не отдан наружу — безопасно падать здесь (`IllegalStateException`), если конфигурация невалидна, чем ронять приложение позже на реальном запросе.
>
> **3. Custom init-method — `@Bean(initMethod = "...")`**
> Нужен только для **чужого класса**, в который нельзя добавить `@PostConstruct`.
> ```java
> @Bean(initMethod = "connect")
> public LegacyFtpClient ftpClient() {
>     return new LegacyFtpClient("ftp.example.com");
> }
> ```
> Зачем: `LegacyFtpClient` — из внешней библиотеки, аннотацию туда не добавить, а метод `connect()` вызвать после создания бина как-то надо.
>
> **4. `@PreDestroy`** (не срабатывает при @Scope("prototype"))
> Освобождение ресурсов: закрыть соединения, остановить executor/поток, сбросить буферы, отписаться от подписок.
> ```java
> @PreDestroy
> public void shutdown() {
>     httpClient.close();
> }
> ```
> Зачем: гарантированно вызывается при `context.close()` / штатном graceful shutdown — без этого утечка соединений/файловых дескрипторов при перезапуске приложения.
>
> **5. Custom destroy-method — `@Bean(destroyMethod = "...")`**
> Аналог `@PreDestroy` для чужого класса.
> ```java
> @Bean(destroyMethod = "close")
> public HikariDataSource dataSource() { ... }
> ```
> Зачем: `HikariDataSource` не ваш класс — но Spring даже без явного `destroyMethod` сам находит и вызывает `close()`/`shutdown()` (inferred destroy method).
>
> **6. `SmartInitializingSingleton.afterSingletonsInstantiated()` — бонус**
> Единственный официальный способ дождаться, что **вообще все** non-lazy синглтоны контекста созданы (не только свои прямые зависимости, как в `@PostConstruct`). При @Scope("prototype") как правило не срабатывает. 
> ```java
> @Component
> public class StartupChecker implements SmartInitializingSingleton {
>     public void afterSingletonsInstantiated() {
>         // финальная проверка/связка между независимыми бинами
>     }
> }
> ```
> Зачем: если нужна логика, которая требует существования бинов, не являющихся прямыми зависимостями текущего — `@PostConstruct` для этого не подходит (см. пример с `@Lazy`-бином, который ещё не создан).

>[!question]- какой метод срабатывает после инициализации всех бинов
>run , чтобы его использоватжь надо переопределить его в классе  помеченном @component и реализовать интерефейс ComandLineRunner

>[!question]- какой этап инициализации бина можно использовать для логирования и как это сделать через аспект
> Ни `@PostConstruct`, ни `postProcessBeforeInitialization`/`postProcessAfterInitialization` **не подходят** для "залогировать каждый вызов метода бина" — все они срабатывают **один раз**, при создании бина, а не при каждом вызове `method1()`/`method2()`.
>
> `postProcessAfterInitialization` можно использовать только чтобы **один раз настроить** перехват на будущее — обернуть бин в прокси с `MethodInterceptor`. Сам лог пишется не в теле `postProcessAfterInitialization`, а внутри лямбды интерцептора — она и выполняется на каждый реальный вызов:
> ```java
> @Override
> public Object postProcessAfterInitialization(Object bean, String beanName) {
>     if (bean instanceof EventFightersService) {
>         ProxyFactory pf = new ProxyFactory(bean);
>         pf.addAdvice((MethodInterceptor) invocation -> {
>             long start = System.nanoTime();
>             Object result = invocation.proceed();
>             log.info("{} выполнился за {} мкс", invocation.getMethod().getName(),
>                     (System.nanoTime() - start) / 1000);
>             return result;
>         });
>         return pf.getProxy(); // подмена бина на прокси — происходит один раз
>     }
>     return bean;
> }
> ```
>
> Но правильный, идиоматичный инструмент для этой задачи — **`@Aspect`**, а не ручной `BeanPostProcessor`:
> ```java
> @Aspect
> @Component
> public class TimingAspect {
>     @Around("execution(* org.example.springbeans.bean.*.EventFightersService.*(..))")
>     public Object logTiming(ProceedingJoinPoint pjp) throws Throwable {
>         long start = System.nanoTime();
>         try {
>             return pjp.proceed();
>         } finally {
>             log.info("{} выполнился за {} мкс", pjp.getSignature().getName(),
>                     (System.nanoTime() - start) / 1000);
>         }
>     }
> }
> ```
> Pointcut `execution(* ...EventFightersService.*(..))` покрывает **все** методы класса, а тело `@Around` выполняется на **каждый** реальный вызов. Это по сути тот же механизм (`AbstractAutoProxyCreator` — тоже `BeanPostProcessor` под капотом), просто без ручного написания прокси-обёртки.

>[!question]- Fail-fast валидация конфигурации через `BeanPostProcessor` (проверено запуском)
> Пример реального (не библиотечного) use-case для своего `BeanPostProcessor`: проверить ВСЕ бины с определённым маркером ещё во время старта контекста — до того, как приложение начнёт принимать трафик.
>
> Маркер + бин:
> ```java
> @Retention(RetentionPolicy.RUNTIME)
> @Target(ElementType.TYPE)
> public @interface ExternalEndpoint {
>     String url();
> }
>
> @Component
> @ExternalEndpoint(url = "https://api.example.com/pay")
> public class PaymentGatewayClient { }
> ```
>
> Валидатор:
> ```java
> @Component
> public class ExternalEndpointValidator implements BeanPostProcessor {
>     @Override
>     public Object postProcessBeforeInitialization(Object bean, String beanName) {
>         ExternalEndpoint a = bean.getClass().getAnnotation(ExternalEndpoint.class);
>         if (a != null && a.url().isBlank()) {
>             throw new IllegalStateException(
>                 "Бин '%s' помечен @ExternalEndpoint, но url не задан".formatted(beanName));
>         }
>         return bean;
>     }
> }
> ```
>
> Реальный результат запуска с пустым `url = ""`:
> ```
> BeanCreationException: Error creating bean with name 'brokenClient' ...
> Caused by: IllegalStateException: Бин 'brokenClient' помечен @ExternalEndpoint, но url не задан
> 	at ExternalEndpointValidator.postProcessBeforeInitialization(...)
> 	at AbstractAutowireCapableBeanFactory.applyBeanPostProcessorsBeforeInitialization(...)
> ```
> Исключение, брошенное внутри `postProcessBeforeInitialization`, не проглатывается — Spring оборачивает его в `BeanCreationException` и **останавливает весь `refresh()` контекста**. Приложение не поднимется вообще — ошибка конфигурации обнаружена за секунды при старте, а не через час в проде на первом реальном вызове.
>
> Нюанс: атрибуты аннотаций — compile-time константы, `@ExternalEndpoint(url = "${payment.url}")` сам по себе не резолвится как `@Value`. Если нужно тянуть `url` из `application.yaml`, валидатору нужно самому резолвить плейсхолдер через `Environment.resolvePlaceholders(...)`.

>[!question]-  Название бинов могут совпадать ? 
> В пределах одного контекста нет -   например в  ApplicationContext  , избавиться от дублей можно через @Bean(name = "bean1") . **В разных `ApplicationContext`** (например, в родителе и потомке) имена могут совпадать. 

>[!question]-  как получить все бины контекста
> ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig1.class);
> String[] beanNames = context.getBeanDefinitionNames(); 

>[!question]- импортировать один конфиг в другой 
> @Configuration
  @Import(AppConfig2.class)

>[!question]- @Configuration
>ОБЕСПЕЧИВАЕТ СИНГЛТОН  СВОИХ БИНОВ  - обеспечивает создание прокси класса конфигурации, благодаря чему вызовы методов с @Bean при инициализации других бинов возвращают управляемые синглтон-бины из контекста. Без @Configuration таких гарантий нет, и каждый метод с @Bean вызывается как обычный метод, создающий новый объект.

>[!question]- BeanFactory
>Базовый IoC контейнер — управляет созданием, хранением и выдачей бинов
>`BeanFactory` — это как "DI-двигатель" под капотом Spring.

>[!question]- ApplicationContext
>Расширяет `BeanFactory`, добавляет всё, что нужно в реальном приложении 

>[!question]- @Primary
>приоритет бина -  указывает, что данный bean должен быть приоритетным (основным) при внедрении зависимостей, когда существует несколько bean-ов одного типа.  
>без него ошибка 
>```
>NoUniqueBeanDefinitionException: expected single matching bean but found 2
>```
>пример 
>```
>public class EnglishGreetingService implements GreetingService {
>...
>public class RussianGreetingService implements GreetingService {
>...
>public interface GreetingService {
>```

>[!question]- @Component
> внедряется автоматчиески при  запуске проги с помощью  @ComponentScan . его потомки(`@Service`, `@Repository`, `@Controller` и т.д.)

>[!question]- @Qualifier("russianGreetingService") 
>указывает какой бин использовать если дубль бина интерфейса
>```
>@Service
public class RussianGreetingService implements GreetingService {
>```

>[!question]-  @ComponentScan это 
>**отвечает за поиск и регистрацию бинов**, созданных через аннотации вроде `@Component`, `@Service`, `@Repository`, `@Controller`  `@Configuration` и т. д. 
>просканирует все бины не зависимо от того над кем навешана аннотация 
>входит в  @SpringBootApplication  
>не сканирует @Bean если он не находится в `@Configuration` классе 

>[!question]- @SpringBootApplication
>  =  @Configuration
@EnableAutoConfiguration
@ComponentScan 
> можно укзать пакет сканирования   @SpringBootApplication(scanBasePackages = "com.example.testlinux") 

>[!question]- @Bean зависимость когда содается и может ли находится в классе без аннотации бина
> Если метод с аннтоацией @Bean находится в классе не помченном ни какой аннотации - то спринг его не увидит вообще (но достучаться до него можно через регистрацию его в контексте внутри метода main или через @Configuration @Import(HiddenConfig.class), а если помечен - то спринг просканирет его при старте приложения . 

>[!question]-  singleton может хранить состояние  ? 
> Да, **синглтон может хранить состояние**  

>[!question]- @Lazy 
>линивая инициализация бина при первом обращение к нему во время выполнения проги - пример 
>```
>@Bean 
>@Lazy
> public someMethod(){}
>```

## Область видимости бина 
>[!question]- scope (область видимости бина)
>**singleton** - один объект - инстанс класса на весь контейнер , примеры - сервисы, репозитории
>```
>@Component @Scope("singleton") public class MyService { }
>```
>**prototype** - Каждый запрос к контейнеру (`getBean()`) создаёт **новый объект**
>Полезно для stateful-объектов (например, временных DTO, каких-то worker-объектов).
>```
>@Component @Scope("prototype") public class MyService { }
>```
> **request** (веб скоуп) - работает только в WebApplicationContext 
>один бин на каждый http запрос
>после завершения запроса бин уничтожается 
>Применяется для хранения данных запроса (например, user context). 
>```
>@Component @Scope("request") public class MyService { }
>```
>**session** Один бин на HTTP-сессию пользователя.
> ```
>@Component @Scope("session") public class MyService { }
>```
>**application** один бин на все приложение (обычно совпадает с singleton, но на уровне веб-приложения). 
>```
>@Component @Scope("application") public class MyService { }
>```
>**websocket** один бин на вебсокет сессию 
>```
>@Component @Scope("application") public class MyService { }
>```

>[!question]- Время жизни сессии
>Когда клиент (браузер) впервые обращается к серверу, сервер может создать **HTTP-сессию**.
>Сервер присваивает ей **уникальный идентификатор (session ID)** и отсылает его клиенту в cookie (`JSESSIONID`).
>При последующих запросах браузер отправляет это cookie обратно, и сервер понимает: "ага, это тот же пользователь, у него уже есть сессия".
>Сессия хранится на сервере и имеет **таймаут бездействия**.
>По умолчанию в Spring Boot (через embedded Tomcat) — **30 минут**.
>Таймаут можно изменить в `application.properties` / `application.yml`:
>

>[!question]- У приложения может быть несколько ioc контейнеров ?
>Да - пример 
>```
>public class TestBean {  
>  
>    public static void main(String[] tons) {  
>        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig1.class);  
>        MyBean bean1 = (MyBean) context.getBean("bean1");  
>        bean1.sayHello();  
>  
>        ApplicationContext context3 = new AnnotationConfigApplicationContext(AppConfig3.class);  
>        MyBean bean2 = (MyBean) context3.getBean("bean1");  
>        bean2.sayHello();  
>    }  
>}
>>---
>
>@Configuration  
>public class AppConfig1 {  
>  
>    @Bean(name = "bean1")  
>    public MyBean myBean() {  
>        return new MyBean(25);  
>    }  
>}
>>---
>@Configuration  
>public class AppConfig3 {  
>  
>    @Bean(name = "bean1")  
>    public MyBean myBean() {  
>        return new MyBean(852);  
>    }  
>}
>```

>[!question]- Какие реализации IoC существуют
>Пиши

>[!question]- Отличие @Component от @Bean
>@Component работает в связке с @ComponentScan и ставится над классом. Используется для своих классов (сервисы репозитории мапперы)
>
>@Bean ставится над методом и говорит контейнеру - этот метод возвращает бин, положи его в контейнер - Для бинов из внешних библиотек или когда нужно вручную контролировать создание объекта|

>[!question]- Если используется scope singleton то когда лучше использовать статические утилитные классы, а когда бины 
>когда нет зависмостей от репозитория и других зависимостей спринга - нет состояния которое нужно хранить с помощью этих зависимостей 

