>[!question]- scope (область видимости бина)
> Рецепт создания объекта бина?
>**singleton** - один объект - инстанс класса на весь контейнер , примеры - сервисы, репозитории
>```
>@Component @Scope("singleton") public class MyService { }
>```
>**prototype** - Каждый запрос к контейнеру (`getBean()`) создаёт **новый объект**
>Полезно для stateful-объектов (например, временных DTO, каких-то worker-объектов).
>```
>@Component @Scope("pretotype") public class MyService { }
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

## Создание и жизненный цикл бинов

>[!question]- какой метод срабатывает после инициализации всех бинов
>run , чтобы его использоватжь надо переопределить его в классе  помеченном @component и реализовать интерефейс ComandLineRunner

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

>[!question]-  Порядок инициализации бина 
> Чиcтые бины инициализируются позже тех, в которые они внедренны 
> Сначала создаются бины, от которых зависят другие 
> Spring строит граф зависимостей 
> 1. Обнаружение компонента - логировать через бин не возможно 
> 2. Вызов конструктора 
> 3. BeanPostProcessor.postProcessBeforeInitialization 
> 4. Вызов @PostConstruct
> 5. InitializingBean.afterPropertiesSet  - Первая точка, где Spring ГАРАНТИРОВАННО внедрил все зависимости.
> 6. BeanPostProcessor.postProcessAfterInitialization

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

>[!question]- @Bean зависимость когда содается
> Если метод с аннтоацией @Bean находится в классе не помченном ни какой аннтоацией - то спринг его не увидит вообще (но достучаться до него можно через регистрацию его в контексте внутри метода main или через @Configuration @Import(HiddenConfig.class), а если помечен - то спринг просканирет его при старте приложения . 

>[!question]-  singleton может хранить состояние  ? 
> Да, **синглтон может хранить состояние**  

>[!question]- @Lazy 
>линивая инициализация бина при первом обращение к нему во время выполнения проги - пример 
>```
>@Bean 
>@Lazy
> public someMethod(){}
>```

