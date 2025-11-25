# Бины
1. [x] что можно сделать на спринг и что делают (мобильные приложения, десктопные приложения, веб приложения ... )
2. [ ] благодаря поддержке транзакций, интеграции с JMS, Kafka и другими сообщениями, Spring активно используется для создания банковских систем, страхования, складских систем и документооборота.
3. [x] напиши прмер приложения с веб запросами , без @SpringBootApplication 
4. [x] может ли @Configuration класс просканировать бины в других пакетах с помощью @ComponentScan - да
5. [x] в каком порядке инициализируются бины - ДОПИШИ здесь и пример  с инициализацией и логами  18.05 - 19.41
6. [ ] возможно ли подлкючиться к бд без application.properties
7. [ ] встроенный серверы в Spring boot - Tomcat, Jetty 
8. [ ] отличие @GetMapping("/hello") от @RequestMapping(value = "/hello", method = RequestMethod.GET)
9. [ ] AOP в Spring 
10. [ ] как создается изолирванный конитекст 
11. [ ] hikary poll 
12. [ ] Подключения к БД:  credentials, pool settings
13. [ ] елк стек, актуатор , графана 

>[!question]-  Порядок инициализации бина 
> Читые бины инициализируются позже тех, в которые они внедренны 
> Сначала создаются бины, от которых зависят другие 
> Spring строит граф зависимостей 
> 1. Обнаружение компонента - логировать через бин не возможно 
> 2. Вызов конструктора 
> 3. BeanPostProcessor.postProcessBeforeInitialization 
> 4. Вызов @PostConstruct
> 5. InitializingBean.afterPropertiesSet  - Первая точка, где Spring ГАРАНТИРОВАННО внедрил все зависимости.
> 6. BeanPostProcessor.postProcessAfterInitialization

>[!question]- предназначение 
>>Spring — это мощный и гибкий фреймворк для разработки приложений на Java, который применяется в самых разных сферах:  
>  
>Веб-приложения и REST API: с помощью Spring MVC и Spring WebFlux можно создавать как классические, так и реактивные веб-приложения, обрабатывать HTTP-запросы, генерировать страницы на сервере, реализовывать клиент-серверные сервисы.  
>  
>Микросервисы: Spring Boot и Spring Cloud предоставляют средства для быстрого построения микросервисной архитектуры с автоконфигурацией, сервис-дискавери, маршрутизацией, балансировкой нагрузки.  
>  
>Корпоративные системы: благодаря поддержке транзакций, интеграции с JMS, Kafka и другими сообщениями, Spring активно используется для создания банковских систем, страхования, складских систем и документооборота.  
>  
>Работа с базами данных: Spring Data упрощает запросы к реляционным и NoSQL базам, управление транзакциями и репозиториями.  
>  
>Консольные и фоновые приложения: Spring позволяет создавать надежные серверные и batch-приложения для автоматизации бизнес-процессов.  
>  
>        Безопасность: Spring Security обеспечивает комплексную аутентификацию, авторизацию и защиту приложений.  
>  
>При этом Spring применяется преимущественно для серверной части (backend). Для мобильных и десктопных приложений он используется редко, обычно в связке с другими технологиями.  
>  
>        Итог: Spring — это универсальный фреймворк для построения масштабируемых, надежных и легко управляемых серверных приложений в самых разных сферах, особое внимание уделяется веб-разработке, микро-сервисам и корпоративным системам.

>[!question]-  Название бинов могут совпадать ? 
> В пределах одного контекста нет -   например в  ApplicationContext  , избавиться от дублей можно чеоез @Bean(name = "bean1") 

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

>[!question]- @Bean когда создается 
> при обращение к методу ? 

>[!question]-  singleton может хранить состояние  ? 
> Да, **синглтон может хранить состояние**  

>[!question]- @Lazy 
>линивая инициализация бина при первом обращение к нему во время выполнения проги 
# Остальное 

>[!question]- добавить jpa
>```xml
><dependency>
>    <groupId>org.springframework.boot</groupId>
>    <artifactId>spring-boot-starter-data-jpa</artifactId>
></dependency>
>
><dependency>
>    <groupId>com.mysql</groupId>
>    <artifactId>mysql-connector-j</artifactId>
>    <scope>runtime</scope>
></dependency>
>```

>[!question]- логи статистики jpa/hibernate
> spring.jpa.properties,hubernate.generate_statistics=false
> spring.jpa.properties.hibernate.generate_statistics=true

>[!question]- запустить спринг проект
>mvn spring-boot:run

>[!question]- если @Transactionl не используется то в каких методах не нужен .save
>```List<Battle> battles = battlesRepository.getBattlesFromClient(battleIdList); 
>for  (Battle battle : battles) {  
 >   battlesRepository.delete(battle);  
>}
>```

>[!question]- как в ij добавить данные из дампа БД
>удалить полностью в гуи localhost
>создать бд через терминал в докере 
>```
>create database co34818_sign
>``` 
>запустить скрипт через в контекстном меню run sql script правой кнопкокй по localhost 

>[!question]- Spring hateoas
>библиотека, которая позволяет добавлять в  рест-ответы ссылки на возможные действия 
>```{
  "id": 1,
  "name": "Alex",
  "_links": {
>"self": { "href": "http://localhost:8080/users/1" },
>"all": { "href": "http://localhost:8080/users" }
  }
}
>```
>подходит когда 
>**публичных API**, где важно самоописание (например, GitHub API, PayPal API);
>когда клиентам не известен весь список урл адресов
>бизнес логика часто менятея и API должен подсказывать клиентучто делать 

>[!question]- **Spring WebFlux**
>потоки не блокируются и могут переиспользоваться при запросах
>больше подходит для стриминга

