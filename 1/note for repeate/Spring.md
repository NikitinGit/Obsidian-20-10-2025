# Задачи
1. [x] что можно сделать на спринг и что делают (мобильные приложения, десктопные приложения, веб приложения ... )
2. [ ] благодаря поддержке транзакций, интеграции с JMS, Kafka и другими сообщениями, Spring активно используется для создания банковских систем, страхования, складских систем и документооборота.
3. [x] напиши прмер приложения с веб запросами , без @SpringBootApplication 
4. [x] может ли @Configuration класс просканировать бины в других пакетах с помощью @ComponentScan - да
5. [x] в каком порядке инициализируются бины - ДОПИШИ здесь и пример  с инициализацией и логами  18.05 - 19.41
6. [x] чем отличается @Bean  созданный внутри @Configuration класса от просто внутри класса ?- ДОПИШИ пример из чатджпт 
7. [x] что такое CGLIB 
8. [x] Можно ли использовать @Entity как класс с полями , а действия над ними (бизнес логику) проводить в другом классе который от него наследуется - нормальный ли это подход ? Или обычно методы бизнес логики назодятся в том же @Entity  классе ? - нет
9. [x] Попробуй jOOQ https://www.jooq.org/ 
10. [x] Когда происходит не явный inner join при выборке - допиши возникает ли если c.country это не сущность а int 
11. [ ] jpql к чему относится в  хайбернате
12. [ ] в mysql 5.7 нужно сохранить длительность боя в часах минутах секундах, какой должен быть тип этого поля  - в секундах, достаточно integer , на java исползуется Duration  - напиши памятку 
13. [ ] обязательно ли указывать в hibernate timezone и как принято хранить время с чаовым поясом в Spring (например фронт отправляет время и часовой пояс какого то события, а так же время с датой его  отправки ) - желательно - напиши памятку 
14. [ ] динамический прокси-объект - существует ли статический - да , допиши 
15. [ ] интерсептор и мидлвеар - допиши
16. [ ] детальнее про фабричный метод патерн - почему @Bean это он
17. [ ] Проблемы с Hibernate прокси:- как хайбернейт создает проки объект в которм equals | hashcode не работает - 
18. [ ] Безопасно ли при выборке делать сравнение с объектом сущостью или лучше с id - List\<JudgeScore> findAllByBattleAndJudge(Battle battle, Judge judge); \-  когда могут возникнуть проблеммы ?
19. [ ]  Почему в JudgeScoreRepository можно делать выборку других сущностей не относящихся к этому репозиторию и правильно ли так делать 
20. [ ] Прочитай https://docs.spring.io/spring-framework/reference/web.html и сравни со своим проектом
21. [x] Попробуй вариант Решение 2: QueryDSL (IDE видит usage!) в клоде - который должен показывать usage  
22. [ ] как кэшировать гет запросы 
23. [ ] Domain-Driven Design (Доменно-ориентированное проектирование)  
24. [ ] Всегда ли есть смысл делать связь на уровне ентити а не только на уровне БД ?
25. [ ] какие аннотации надо знать на собеседовании 
26. [ ] @Profile
27. [ ] @ConditionProperty
28. [ ] КЭШ первого и второго уровня
29. [ ] возможно ли подлкючиться к бд без application.properties
30. [ ] как запустить не ddl миграции в спринг - посмотри в MigrationService.java 
31. [ ] встроенный серверы в Spring boot - Tomcat, Jetty 
32. [ ] отличие @GetMapping("/hello") от @RequestMapping(value = "/hello", method = RequestMethod.GET)
33. [ ] AOP в Spring 
34. [ ] Как создается изолирванный контекст 
35. [ ] hikary poll 
36. [ ] Подключения к БД:  credentials, pool settings
37. [ ] елк стек, актуатор , графана 
# Основы 
>[!question]- как принято хранить длительность события 
>обычно хранят в integer  а чтобы в сущности оно выглядело более читаемым используется Duration и конвертируется в Integer с  помощью @Convert

>[!question]- отличие динамического прокси от аннтоации @Convert
>@Convert срабатыывает при  JDBC read/**write**  , прямой вызов конвертера (еще пример  @Enumerated )
>AOP при вызове метода бина , перехвать через прокси (примеры  @Transactional, @Cacheable )

>[!question]- AopUtils
> в нем находятся ошибки и чаще всего вызываются при ошибках в таблицах , отстутсвие таблицы в БД 

>[!question]- Когда происходит не явный inner join при выборке 
>```
>@Query("SELECT new com.strikerstat.webapp.dto.olympic_events.CityDto(c.country, c.region) "
>```
> и country - это объект класса Country, если он null - то в  выборку не попадает
> Работает только с сущностями - если просто поля объекта = null  то inner join не происходит, например 
> ```
> SELECT new com.strikerstat.webapp.dto.olympic_events.CityDto(c.id, c.name)
> ```

>[!question]- какой метод срабатывает после инициализации всех бинов
>run , чтобы его использоватжь надо переопределить его в классе  помеченном @component и реализовать интерефейс ComandLineRunner

>[!question]- `CGLIB` это
>Code Generation Library
> билиотека динамического создания классов и проксиобъектов в рантайме 
> - подменять вызовы методов
создавать прокси для @Configuration классов
делать AOP (аспектно-ориентированное программирование)
внедрять транзакции (@Transactional)
реализовывать lazy loading
>- создавать бины без интерфейсов

>[!question]- `@Bean` это
> своего рода фабричный метод - ставится только над методами 
> без @Configuration над классом в котором находится этот бин Методы @Bean НЕ ПРОКСИРУЮТСЯ через CGLIB -  без @Configuration   возможна ошибка нарушения синглтон типа SELF-INVOKED CALL  - внедрение помеченной @Bean зависимости в конструктор 

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
> В пределах одного контекста нет -   например в  ApplicationContext  , избавиться от дублей можно через @Bean(name = "bean1") 

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

>[!question]- добавить jpa в maven .pom
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

>[!question]- Можно ли использовать @Entity как класс с полями , а действия над ними (бизнес логику) проводить в другом классе который от него наследуется - нормальный ли это подход ? Или обычно методы бизнес логики назодятся в том же @Entity  классе ?
> НЕТ, наследование от @Entity для бизнес-логики — это НЕ рекомендуемый подход. Вот почему:
> Hibernate создаст дополнительные таблицы/колонки
> Стратегии @Inheritance (SINGLE_TABLE, JOINED, TABLE_PER_CLASS) усложняют схему БД
> Запросы становятся медленнее из-за JOIN'ов
> Проблемы с Hibernate прокси:
> Hibernate создает прокси-объекты для lazy loading
> Наследник может сломать механизм прокси
> ClassCastException при работе с прокси
> Можно ;
> 1. Anemic Domain Model (рекомендуется для большинства случаев):
> 2. Rich Domain Model (DDD подход):
> 3. Domain-Driven Design (Доменно-ориентированное проектирование) — это подход к разработке, где архитектура строится вокруг бизнес-логики (домена), а не технических деталей.  Основная идея DDD:
>    Код должен отражать бизнес-процессы, а не быть просто "хранилищем данных".

>[!question]- Как избежать дублирования конструктора DTO в JPA запросах?
> **Проблема:** Несколько методов репозитория используют один и тот же длинный конструктор DTO в JPQL
> ```java
> @Query("SELECT new com.example.dto.RoundScoresDto(" +
>        "t.id, t.judge.id, t.battle.idBattle, t.judge.fullName, t.judge.slug, " +
>        "t.fighter1Id, t.fighter1Score, t.fighter2Id, t.fighter2Score, t.roundNumber, t.winnerId) " +
>        "FROM JudgeRoundScore t WHERE t.battle.idBattle IN (:battleIds)")
> List<RoundScoresDto> getAllJudgeRoundScoresListDto(@Param("battleIds") Set<Long> battleIds);
>
> @Query("SELECT new com.example.dto.RoundScoresDto(" +
>        "t.id, t.judge.id, t.battle.idBattle, t.judge.fullName, t.judge.slug, " +
>        "t.fighter1Id, t.fighter1Score, t.fighter2Id, t.fighter2Score, t.roundNumber, t.winnerId) " +
>        "FROM JudgeRoundScore t WHERE t.battle.idBattle = :battleId AND t.judge.id = :judgeId")
> List<RoundScoresDto> getJudgeRoundScoresDto(@Param("battleId") long battleId, @Param("judgeId") int judgeId);
> ```
>
> **Решение через Criteria API + Custom Repository:**
>
> 1. Создать интерфейс кастомного репозитория:
> ```java
> public interface JudgeRoundScoreRepositoryCustom {
>     List<RoundScoresDto> getAllJudgeRoundScoresListDto(Set<Long> battleIds);
>     List<RoundScoresDto> getJudgeRoundScoresDto(long battleId, int judgeId);
> }
> ```
>
> 2. Создать имплементацию с Criteria API:
> ```java
> public class JudgeRoundScoreRepositoryCustomImpl implements JudgeRoundScoreRepositoryCustom {
>     @PersistenceContext
>     private EntityManager entityManager;
>
>     @Override
>     public List<RoundScoresDto> getAllJudgeRoundScoresListDto(Set<Long> battleIds) {
>         CriteriaBuilder cb = entityManager.getCriteriaBuilder();
>         CriteriaQuery<RoundScoresDto> query = cb.createQuery(RoundScoresDto.class);
>         Root<JudgeRoundScore> root = query.from(JudgeRoundScore.class);
>
>         Join<JudgeRoundScore, Judge> judgeJoin = root.join("judge");
>         Join<JudgeRoundScore, Battle> battleJoin = root.join("battle");
>
>         // КОНСТРУКТОР DTO В ОДНОМ МЕСТЕ
>         query.select(cb.construct(
>                 RoundScoresDto.class,
>                 root.get("id"),
>                 judgeJoin.get("id"),
>                 battleJoin.get("idBattle"),
>                 judgeJoin.get("fullName"),
>                 judgeJoin.get("slug"),
>                 root.get("fighter1Id"),
>                 root.get("fighter1Score"),
>                 root.get("fighter2Id"),
>                 root.get("fighter2Score"),
>                 root.get("roundNumber"),
>                 root.get("winnerId")
>         ));
>
>         query.where(battleJoin.get("idBattle").in(battleIds));
>         return entityManager.createQuery(query).getResultList();
>     }
>
>     @Override
>     public List<RoundScoresDto> getJudgeRoundScoresDto(long battleId, int judgeId) {
>         // Та же проекция, но другой WHERE
>         CriteriaBuilder cb = entityManager.getCriteriaBuilder();
>         CriteriaQuery<RoundScoresDto> query = cb.createQuery(RoundScoresDto.class);
>         Root<JudgeRoundScore> root = query.from(JudgeRoundScore.class);
>
>         Join<JudgeRoundScore, Judge> judgeJoin = root.join("judge");
>         Join<JudgeRoundScore, Battle> battleJoin = root.join("battle");
>
>         query.select(cb.construct(/* та же проекция */));
>         query.where(cb.and(
>                 cb.equal(battleJoin.get("idBattle"), battleId),
>                 cb.equal(judgeJoin.get("id"), judgeId)
>         ));
>         return entityManager.createQuery(query).getResultList();
>     }
> }
> ```
>
> 3. Расширить основной репозиторий:
> ```java
> public interface JudgeRoundScoreRepository
>     extends JpaRepository<JudgeRoundScore, Long>, JudgeRoundScoreRepositoryCustom {
>     // Spring автоматически найдет имплементацию по имени *Impl
> }
> ```
>
> **Преимущества:**
> - ✅ Конструктор DTO в одном месте
> - ✅ Типобезопасность (ошибки на этапе компиляции)
> - ✅ Легко рефакторить - изменение DTO требует правки в одном месте
> - ✅ Не требует новых зависимостей (Criteria API - часть JPA)
>
> **Производительность JPQL vs Criteria API:**
> - SQL генерируется одинаковый
> - Разница только в построении запроса: ~0.0001-0.0002 мс
> - Основное время: выполнение SQL в БД (1-100 мс)
> - **Вывод: разница < 0.01% - незаметна**

>[!question]- @PersistenceContext - что это и зачем?
> **@PersistenceContext** - это стандартная JPA аннотация (не Spring!) для инжекта EntityManager
>
> ```java
> import jakarta.persistence.PersistenceContext;  // JPA 3.0 (Jakarta EE)
> // или
> import javax.persistence.PersistenceContext;    // JPA 2.x (старые версии)
>
> @PersistenceContext
> private EntityManager entityManager;
> ```
>
> **Что делает:**
> 1. **Инжектит EntityManager** - главный интерфейс JPA для работы с БД
> 2. **Transaction-scoped** - каждый запрос в транзакции получает свой EntityManager
> 3. **Thread-safe proxy** - Spring создает прокси, безопасный для многопоточности
> 4. **Автоматическое управление** - не нужно вручную создавать/закрывать
>
> **Атрибуты:**
> ```java
> @PersistenceContext(
>     unitName = "myPU",                           // Имя Persistence Unit (если их несколько)
>     type = PersistenceContextType.TRANSACTION    // TRANSACTION (default) или EXTENDED
> )
> private EntityManager entityManager;
> ```
>
> **Альтернативы:**
> ```java
> // Вариант 1: Constructor injection (более современный)
> private final EntityManager entityManager;
> public MyRepo(EntityManager entityManager) {
>     this.entityManager = entityManager;
> }
>
> // Вариант 2: @Autowired (менее предпочтительно)
> @Autowired
> private EntityManager entityManager;
> ```
>
> **Почему @PersistenceContext лучше:**
> - Стандарт JPA (переносимость между провайдерами)
> - Явно показывает намерение (работа с persistence context)
> - Код остается переносимым между JPA-провайдерами

>[!question]- Почему IntelliJ не видит usage конструктора DTO в Criteria API?
> **Проблема:** После рефакторинга на Criteria API, IntelliJ показывает "No usages" для конструктора DTO
>
> **Причина:**
> В Criteria API конструктор вызывается через рефлексию в runtime:
> ```java
> query.select(cb.construct(
>     RoundScoresDto.class,  // ← Класс передается как параметр
>     root.get("id"),
>     // ...
> ));
> ```
> `cb.construct()` принимает `Class<?>` и вызывает конструктор **в runtime через рефлексию**.
> IntelliJ не может статически проанализировать такие вызовы.
>
> **В JPQL (старый вариант) работало:**
> ```java
> @Query("SELECT new com.example.dto.RoundScoresDto(...)")
> ```
> IntelliJ парсит JPQL-строки и находит конструкторы в выражении `new ...()`.
>
> **Решения:**
>
> 1. **Подавить предупреждение (рекомендуется):**
> ```java
> @SuppressWarnings("unused")
> public RoundScoresDto(Long id, Integer judgeId, ...) {
>     // ...
> }
> ```
>
> 2. **Добавить фиктивное использование в тестах:**
> ```java
> @Test
> void constructorUsedByCriteriaAPI() {
>     new RoundScoresDto(1L, 2, 3L, "name", "slug", 4, 5.0, 6, 7.0, 8, 9);
> }
> ```
>
> 3. **Использовать Blaze-Persistence Entity Views** (если уже есть в проекте):
> ```java
> @EntityView(JudgeRoundScore.class)
> public interface RoundScoresDto {
>     Long getId();
>     @Mapping("judge.id")
>     Integer getJudgeId();
>     // ...
> }
> ```
>
> 4. **Вернуться к JPQL + константа:**
> ```java
> private static final String DTO_PROJECTION =
>     "new com.example.dto.RoundScoresDto(t.id, t.judge.id, ...)";
> ```

>[!question]- Criteria API - что это такое?
> **Criteria API** - это **стандартный JPA API** для построения типобезопасных запросов к базе данных через Java код (без строк JPQL)
>
> **Что это:**
> - Часть спецификации JPA (не требует дополнительных библиотек)
> - Позволяет строить запросы программно через объектную модель
> - Альтернатива JPQL строкам
> - Типобезопасность на этапе компиляции
>
> **Основные компоненты:**
> ```java
> // 1. EntityManager - точка входа в JPA
> @PersistenceContext
> private EntityManager entityManager;
>
> // 2. CriteriaBuilder - фабрика для построения запросов
> CriteriaBuilder cb = entityManager.getCriteriaBuilder();
>
> // 3. CriteriaQuery - сам запрос
> CriteriaQuery<Entity> query = cb.createQuery(Entity.class);
>
> // 4. Root - FROM часть (главная таблица)
> Root<Entity> root = query.from(Entity.class);
>
> // 5. Join - JOIN части (связанные таблицы)
> Join<Entity, Related> join = root.join("relatedField");
>
> // 6. Predicate - WHERE условия
> Predicate condition = cb.equal(root.get("field"), value);
>
> // 7. Выполнение
> List<Entity> results = entityManager.createQuery(query).getResultList();
> ```
>
> **Пример простого запроса:**
> ```java
> // JPQL:
> @Query("SELECT t FROM JudgeRoundScore t WHERE t.battle.idBattle = :battleId")
>
> // Criteria API:
> CriteriaBuilder cb = entityManager.getCriteriaBuilder();
> CriteriaQuery<JudgeRoundScore> query = cb.createQuery(JudgeRoundScore.class);
> Root<JudgeRoundScore> root = query.from(JudgeRoundScore.class);
>
> Join<JudgeRoundScore, Battle> battleJoin = root.join("battle");
> query.where(cb.equal(battleJoin.get("idBattle"), battleId));
>
> List<JudgeRoundScore> results = entityManager.createQuery(query).getResultList();
> ```
>
> **Пример с проекцией в DTO:**
> ```java
> CriteriaBuilder cb = entityManager.getCriteriaBuilder();
> CriteriaQuery<RoundScoresDto> query = cb.createQuery(RoundScoresDto.class);
> Root<JudgeRoundScore> root = query.from(JudgeRoundScore.class);
>
> Join<JudgeRoundScore, Judge> judgeJoin = root.join("judge");
> Join<JudgeRoundScore, Battle> battleJoin = root.join("battle");
>
> // Маппинг на конструктор DTO
> query.select(cb.construct(
>     RoundScoresDto.class,
>     root.get("id"),
>     judgeJoin.get("id"),
>     battleJoin.get("idBattle"),
>     judgeJoin.get("fullName"),
>     judgeJoin.get("slug"),
>     root.get("fighter1Id"),
>     root.get("fighter1Score"),
>     root.get("fighter2Id"),
>     root.get("fighter2Score"),
>     root.get("roundNumber"),
>     root.get("winnerId")
> ));
>
> query.where(battleJoin.get("idBattle").in(battleIds));
> return entityManager.createQuery(query).getResultList();
> ```
>
> **Преимущества:**
> - ✅ **Типобезопасность** - ошибки на этапе компиляции
> - ✅ **Стандарт JPA** - не нужны дополнительные библиотеки
> - ✅ **Переносимость** - работает с любым JPA провайдером (Hibernate, EclipseLink, etc)
> - ✅ **Динамические запросы** - легко добавлять условия программно
> - ✅ **Рефакторинг** - IDE видит использование полей
>
> **Недостатки:**
> - ❌ **Многословный** - много кода для простых запросов
> - ❌ **Сложно читать** - не похож на SQL
> - ❌ **Нет автокомплита для полей** - нужно знать точные имена строками
> - ❌ **Проблема с "Find Usages"** - `root.get("field")` - строка, IDE не видит usage
>
> **Когда использовать:**
> - ✅ Динамические запросы с условными фильтрами
> - ✅ Сложная бизнес-логика в репозиториях
> - ✅ Не хочется добавлять QueryDSL
> - ✅ Нужна типобезопасность без сторонних библиотек
> - ✅ Переиспользование общей логики проекций

>[!question]- QueryDSL - что это такое?
> **QueryDSL** = **Query Domain Specific Language** (Язык специфичный для предметной области запросов)
>
> **Что это:**
> - Библиотека для построения типобезопасных SQL/JPQL запросов через Java код
> - Генерирует Q-классы на основе ваших Entity классов
> - Альтернатива JPQL (строки) и Criteria API (многословный)
>
> **Зависимости для Spring Boot:**
> ```xml
> <!-- QueryDSL JPA -->
> <dependency>
>     <groupId>com.querydsl</groupId>
>     <artifactId>querydsl-jpa</artifactId>
>     <classifier>jakarta</classifier>
> </dependency>
>
> <!-- Процессор для генерации Q-классов -->
> <dependency>
>     <groupId>com.querydsl</groupId>
>     <artifactId>querydsl-apt</artifactId>
>     <classifier>jakarta</classifier>
>     <scope>provided</scope>
> </dependency>
> ```
>
> **Пример использования:**
> ```java
> // Вместо JPQL строк:
> @Query("SELECT t FROM JudgeRoundScore t WHERE t.battle.idBattle = :battleId")
>
> // Или Criteria API:
> CriteriaBuilder cb = em.getCriteriaBuilder();
> CriteriaQuery<JudgeRoundScore> query = cb.createQuery(JudgeRoundScore.class);
> Root<JudgeRoundScore> root = query.from(JudgeRoundScore.class);
> query.where(cb.equal(root.get("battle").get("idBattle"), battleId));
>
> // QueryDSL выглядит так:
> QJudgeRoundScore score = QJudgeRoundScore.judgeRoundScore;
> List<JudgeRoundScore> results = queryFactory
>     .selectFrom(score)
>     .where(score.battle.idBattle.eq(battleId))
>     .fetch();
> ```
>
> **Преимущества QueryDSL:**
> - ✅ **Типобезопасность** - ошибки на этапе компиляции
> - ✅ **IDE поддержка** - автокомплит, рефакторинг, "Find Usages" работают
> - ✅ **Читаемость** - код похож на SQL, но на Java
> - ✅ **Нет магических строк** - нельзя опечататься в имени поля
>
> **Недостатки:**
> - ❌ Требует настройки annotation processor
> - ❌ Генерирует дополнительные Q-классы (но в target/, не в git)
> - ❌ Увеличивает размер проекта
> - ❌ Нужно пересобирать после изменения Entity
>
> **Сравнение трех подходов:**
>
> | Подход | Типобезопасность | IDE Support | Читаемость | Многословность |
> |--------|------------------|-------------|------------|----------------|
> | **JPQL** | ❌ (строки) | ⚠️ (ограниченно) | ✅ | ✅ |
> | **Criteria API** | ✅ | ✅ | ❌ | ❌ (очень многословно) |
> | **QueryDSL** | ✅ | ✅ | ✅ | ✅ |
>
> **Когда использовать:**
> - **JPQL** - простые запросы, не хочется доп. зависимостей
> - **Criteria API** - динамические запросы без QueryDSL, нужен Java API стандарт JPA
> - **QueryDSL** - сложные запросы, большой проект, важна типобезопасность и читаемость
>
> **Пример репозитория с QueryDSL:**
> ```java
> // Интерфейс наследует от QuerydslPredicateExecutor
> public interface JudgeRoundScoreRepository
>     extends JpaRepository<JudgeRoundScore, Long>,
>             QuerydslPredicateExecutor<JudgeRoundScore> {
> }
>
> // Использование:
> QJudgeRoundScore score = QJudgeRoundScore.judgeRoundScore;
> Predicate predicate = score.battle.idBattle.in(battleIds)
>     .and(score.judge.id.eq(judgeId));
>
> List<JudgeRoundScore> results = repository.findAll(predicate);
> ```

>[!question]- jOOQ - что это такое?
> **jOOQ** = **Java Object Oriented Querying** (Объектно-ориентированные запросы на Java)
> Коммит с примером https://bitbucket.org/strikerstat/strikerstat/commits/72c197d3619e400ad7800f108621598afc787b08 
> **Что это:**
> - Database-first библиотека для построения типобезопасных SQL запросов через Java код
> - Генерирует Java классы на основе **реальной схемы БД** (не Entity классов!)
> - Предоставляет **ПОЛНУЮ** типобезопасность на этапе компиляции
> - Fluent API близкий к SQL
>
> **Ключевое отличие от QueryDSL:**
> - **QueryDSL**: Code-first (генерирует Q-классы из Java Entity)
> - **jOOQ**: Database-first (генерирует классы из таблиц БД)
>
> **Настройка Maven:**
> ```xml
> <!-- Зависимости -->
> <dependency>
>     <groupId>org.springframework.boot</groupId>
>     <artifactId>spring-boot-starter-jooq</artifactId>
> </dependency>
> <dependency>
>     <groupId>org.jooq</groupId>
>     <artifactId>jooq</artifactId>
>     <version>3.19.1</version>
> </dependency>
>
> <!-- Maven plugin для генерации классов из БД -->
> <plugin>
>     <groupId>org.jooq</groupId>
>     <artifactId>jooq-codegen-maven</artifactId>
>     <version>3.19.1</version>
>     <configuration>
>         <jdbc>
>             <driver>com.mysql.cj.jdbc.Driver</driver>
>             <url>jdbc:mysql://localhost:3306/my_database</url>
>             <user>root</user>
>             <password>password</password>
>         </jdbc>
>         <generator>
>             <database>
>                 <name>org.jooq.meta.mysql.MySQLDatabase</name>
>                 <includes>judges_round_scores|judges|battles</includes>
>                 <inputSchema>my_database</inputSchema>
>             </database>
>             <target>
>                 <packageName>com.example.jooq.generated</packageName>
>                 <directory>target/generated-sources/jooq</directory>
>             </target>
>         </generator>
>     </configuration>
>     <executions>
>         <execution>
>             <phase>generate-sources</phase>
>             <goals>
>                 <goal>generate</goal>
>             </goals>
>         </execution>
>     </executions>
> </plugin>
> ```
>
> **Генерация классов:**
> ```bash
> mvn clean generate-sources  # Сгенерирует классы в target/generated-sources/jooq
> ```
>
> **Пример использования:**
> ```java
> import static com.example.jooq.generated.tables.Judges.JUDGES;
> import static com.example.jooq.generated.tables.JudgesRoundScores.JUDGES_ROUND_SCORES;
>
> @Repository
> @RequiredArgsConstructor
> public class JudgeRoundScoreRepositoryCustomImpl {
>     private final DSLContext dsl;  // Инжектится Spring Boot автоматически
>
>     public List<RoundScoresDto> getAllJudgeRoundScoresListDto(Set<Long> battleIds) {
>         return dsl
>             .select(
>                 JUDGES_ROUND_SCORES.ID,              // ← Типобезопасное поле
>                 JUDGES.ID,
>                 JUDGES_ROUND_SCORES.BATTLE_ID,
>                 JUDGES.FULLNAME,                     // ← Если напишу FULLNAME2 - НЕ СКОМПИЛИРУЕТСЯ!
>                 JUDGES.SLUG,                         // ← Если напишу SLUG2 - НЕ СКОМПИЛИРУЕТСЯ!
>                 JUDGES_ROUND_SCORES.FIGHTER_1_ID,
>                 JUDGES_ROUND_SCORES.FIGHTER_1_SCORE,
>                 JUDGES_ROUND_SCORES.FIGHTER_2_ID,
>                 JUDGES_ROUND_SCORES.FIGHTER_2_SCORE,
>                 JUDGES_ROUND_SCORES.ROUNDNUMBER,
>                 JUDGES_ROUND_SCORES.WINNER_ID
>             )
>             .from(JUDGES_ROUND_SCORES)
>             .join(JUDGES).on(JUDGES_ROUND_SCORES.JUDGE_ID.eq(JUDGES.ID))
>             .where(JUDGES_ROUND_SCORES.BATTLE_ID.in(battleIds))
>             .fetchInto(RoundScoresDto.class);
>     }
> }
> ```
>
> **Интеграция со Spring Data:**
> ```java
> // 1. Создать Custom интерфейс
> public interface JudgeRoundScoreRepositoryCustom {
>     List<RoundScoresDto> getAllJudgeRoundScoresListDto(Set<Long> battleIds);
> }
>
> // 2. Реализация через jOOQ (см. выше)
> public class JudgeRoundScoreRepositoryCustomImpl implements JudgeRoundScoreRepositoryCustom {
>     // jOOQ реализация
> }
>
> // 3. Расширить основной репозиторий
> public interface JudgeRoundScoreRepository
>     extends JpaRepository<JudgeRoundScore, Long>,
>             JudgeRoundScoreRepositoryCustom {  // ← Spring автоматически найдет *Impl
> }
> ```
>
> **Типобезопасность - проверка:**
> ```java
> // ❌ JPQL - скомпилируется, упадет в рантайме
> @Query("SELECT t.judge.slug2 FROM JudgeRoundScore t")
>
> // ⚠️ Criteria API - скомпилируется, упадет в рантайме
> judgeJoin.get("slug2")
>
> // ✅ jOOQ - НЕ СКОМПИЛИРУЕТСЯ, ошибка сразу в IDE!
> JUDGES.SLUG2  // ← Cannot resolve symbol 'SLUG2'
> ```
>
> **Преимущества jOOQ:**
> - ✅ **100% типобезопасность** - все ошибки на этапе компиляции
> - ✅ **Полная поддержка IDE** - автокомплит (Ctrl+Space), рефакторинг (Ctrl+Shift+R), Find Usages (Alt+F7)
> - ✅ **Читаемый код** - fluent API близкий к SQL
> - ✅ **Database-first** - схема БД — единственный источник истины
> - ✅ **Рефакторинг-дружественность** - переименовали колонку в БД → запустили codegen → IDE показывает ВСЕ места использования
> - ✅ **Нет магических строк** - нельзя опечататься в имени поля/таблицы
> - ✅ **Полный контроль SQL** - видно точный SQL в логах
>
> **Недостатки:**
> - ❌ Требует настройки codegen plugin
> - ❌ Нужно перегенерировать при изменении схемы БД
> - ❌ Генерирует много классов (но в target/, не в git)
> - ❌ Database-first подход (если схема БД не контролируется - проблема)
>
> **Когда использовать jOOQ:**
> - ✅ Сложные запросы с множественными JOIN
> - ✅ Запросы с подзапросами и агрегациями
> - ✅ Dynamic queries (условия формируются в runtime)
> - ✅ Когда нужна 100% типобезопасность
> - ✅ Большой проект с множеством запросов
>
> **Когда использовать JPA/JPQL:**
> - ✅ Простые CRUD операции (save, findById, delete)
> - ✅ Работа с entity relationships (@OneToMany, @ManyToOne)
> - ✅ Простые запросы
>
> **Гибридный подход (рекомендуется):**
> Использовать оба инструмента:
> - Spring Data JPA для простых операций
> - jOOQ для сложных запросов
>
> **Сравнение JPQL vs Criteria API vs jOOQ:**
>
> | Критерий | JPQL | Criteria API | jOOQ |
> |----------|------|--------------|------|
> | Типобезопасность | ❌ Нет | ⚠️ Частичная* | ✅ Полная |
> | Рефакторинг | ❌ Не работает | ⚠️ Частично* | ✅ Полностью |
> | Автодополнение | ❌ Нет | ⚠️ Только типы | ✅ Всё |
> | Find Usages в IDE | ❌ Не работает | ❌ Не работает* | ✅ Работает |
> | Читаемость | ✅ Хорошая | ❌ Плохая | ✅ Отличная |
> | Производительность | ✅ Хорошая | ✅ Хорошая | ✅ Отличная |
> | Ошибки компиляции | ❌ Только рантайм | ⚠️ Частично* | ✅ Всё в compile-time |
>
> *В Criteria API имена полей — строки (`get("fieldName")`), поэтому опечатки не находятся на этапе компиляции
>
> **Примеры других операций jOOQ:**
> ```java
> // WHERE с несколькими условиями
> dsl.selectFrom(JUDGES_ROUND_SCORES)
>     .where(
>         JUDGES_ROUND_SCORES.BATTLE_ID.eq(battleId)
>         .and(JUDGES_ROUND_SCORES.ROUNDNUMBER.eq(1))
>         .and(JUDGES_ROUND_SCORES.WINNER_ID.isNotNull())
>     )
>     .fetch();
>
> // GROUP BY и агрегации
> dsl.select(
>         JUDGES_ROUND_SCORES.BATTLE_ID,
>         DSL.count().as("score_count"),
>         DSL.avg(JUDGES_ROUND_SCORES.FIGHTER_1_SCORE).as("avg_score")
>     )
>     .from(JUDGES_ROUND_SCORES)
>     .groupBy(JUDGES_ROUND_SCORES.BATTLE_ID)
>     .fetch();
>
> // ORDER BY + LIMIT
> dsl.selectFrom(JUDGES_ROUND_SCORES)
>     .orderBy(
>         JUDGES_ROUND_SCORES.BATTLE_ID.asc(),
>         JUDGES_ROUND_SCORES.ROUNDNUMBER.desc()
>     )
>     .limit(10)
>     .offset(20)
>     .fetch();
> ```
>
> **Debugging - посмотреть SQL:**
> ```java
> // В application.properties:
> logging.level.org.jooq=DEBUG
>
> // Или в коде:
> String sql = dsl.selectFrom(JUDGES_ROUND_SCORES).getSQL();
> System.out.println(sql);
> ```
>
> **Переиспользование кода (избежать дублирования WHERE):**
>
> Если несколько методов используют одинаковый SELECT + JOIN, но разные WHERE:
>
> ```java
> @Repository
> @RequiredArgsConstructor
> public class JudgeRoundScoreRepositoryCustomImpl {
>     private final DSLContext dsl;
>
>     // Базовый запрос с SELECT и JOIN - переиспользуется во всех методах
>     private SelectJoinStep<Record11<...>> buildBaseQuery() {
>         return dsl
>             .select(ПОЛЯ...)
>             .from(JUDGES_ROUND_SCORES)
>             .join(JUDGES).on(JUDGES_ROUND_SCORES.JUDGE_ID.eq(JUDGES.ID));
>     }
>
>     // Метод 1: свой WHERE
>     public List<RoundScoresDto> method1(Set<Long> battleIds) {
>         return buildBaseQuery()
>             .where(JUDGES_ROUND_SCORES.BATTLE_ID.in(battleIds))
>             .fetchInto(RoundScoresDto.class);
>     }
>
>     // Метод 2: другой WHERE
>     public List<RoundScoresDto> method2(long battleId, int judgeId) {
>         return buildBaseQuery()
>             .where(
>                 JUDGES_ROUND_SCORES.BATTLE_ID.eq(battleId)
>                 .and(JUDGES_ROUND_SCORES.JUDGE_ID.eq(judgeId))
>             )
>             .fetchInto(RoundScoresDto.class);
>     }
>
>     // Метод 3: еще один WHERE - просто добавляем!
>     public List<RoundScoresDto> method3(Set<Long> battleIds, int round) {
>         return buildBaseQuery()
>             .where(
>                 JUDGES_ROUND_SCORES.BATTLE_ID.in(battleIds)
>                 .and(JUDGES_ROUND_SCORES.ROUNDNUMBER.eq(round))
>             )
>             .fetchInto(RoundScoresDto.class);
>     }
> }
> ```
>
> **Преимущества:**
> - ✅ SELECT проекция в одном месте (метод `buildBaseQuery()`)
> - ✅ Легко добавлять новые методы с разными WHERE
> - ✅ Изменение DTO → правка только в `buildBaseQuery()`
> - ✅ НЕ нужно писать новый метод каждый раз!

>[!question]- Как Spring Data JPA находит кастомную реализацию репозитория (паттерн *Impl)?
> **Проблема:** Как добавить собственные методы (jOOQ, Criteria API) к стандартному JpaRepository?
>
> **Решение:** Spring Data JPA использует соглашение о наименовании `*Impl` для автоматического подключения кастомных реализаций
>
> **Структура паттерна:**
> ```java
> // 1. Основной интерфейс репозитория (наследует JpaRepository И Custom интерфейс)
> public interface JudgeRoundScoreRepository
>     extends JpaRepository<JudgeRoundScore, Long>,
>             JudgeRoundScoreRepositoryCustom {  // ← Наследование Custom интерфейса
>
>     // Стандартные JPA методы (findById, save, delete...)
>     boolean isJudgeScoreNotExists(long battleId, int judgeId);
> }
>
> // 2. Custom интерфейс - объявляет дополнительные методы
> public interface JudgeRoundScoreRepositoryCustom {
>     List<RoundScoresDto> getAllJudgeRoundScoresListDto(Set<Long> battleIds);
>     List<RoundScoresDto> getJudgeRoundScoresDto(long battleId, int judgeId);
> }
>
> // 3. Custom реализация - ОБЯЗАТЕЛЬНО суффикс *Impl
> @Repository
> @RequiredArgsConstructor
> public class JudgeRoundScoreRepositoryCustomImpl implements JudgeRoundScoreRepositoryCustom {
>     private final DSLContext dsl;  // jOOQ
>
>     @Override
>     public List<RoundScoresDto> getAllJudgeRoundScoresListDto(Set<Long> battleIds) {
>         return dsl.select(...).from(...).where(...).fetchInto(RoundScoresDto.class);
>     }
>
>     @Override
>     public List<RoundScoresDto> getJudgeRoundScoresDto(long battleId, int judgeId) {
>         return dsl.select(...).from(...).where(...).fetchInto(RoundScoresDto.class);
>     }
> }
> ```
>
> **Как это работает:**
>
> 1. **@Repository на Impl классе:**
>    - Регистрирует класс как Spring-бин
>    - БЕЗ @Repository Spring не найдет эту реализацию
>
> 2. **Соглашение о наименовании (naming convention):**
>    - Spring ищет бин с именем `{ИмяКастомногоИнтерфейса}Impl`
>    - Пример: `JudgeRoundScoreRepositoryCustom` → `JudgeRoundScoreRepositoryCustomImpl`
>    - **ВАЖНО:** Суффикс `Impl` обязателен (можно изменить в настройках, см. ниже)
>
> 3. **Магия Spring Data JPA:**
>    - Spring создает прокси для `JudgeRoundScoreRepository`
>    - Видит наследование `JudgeRoundScoreRepositoryCustom`
>    - Автоматически находит бин `JudgeRoundScoreRepositoryCustomImpl`
>    - **Комбинирует в единый прокси-объект:**
>      - Стандартные JPA методы (findById, save, delete)
>      - Кастомные методы (из JudgeRoundScoreRepositoryCustomImpl)
>
> 4. **Использование:**
> ```java
> @Service
> public class MyService {
>     @Autowired
>     private JudgeRoundScoreRepository repo;  // Получаем единый прокси
>
>     public void example() {
>         // Стандартный JPA метод
>         repo.findById(1L);
>
>         // Кастомный метод (jOOQ)
>         repo.getAllJudgeRoundScoresListDto(battleIds);
>     }
> }
> ```
>
> **Что будет если НЕ использовать суффикс *Impl или назвать по-другому:**
>
> ❌ **Проблема 1: Spring не найдет реализацию**
> ```java
> // ❌ НЕПРАВИЛЬНО - нет суффикса Impl
> @Repository
> public class JudgeRoundScoreRepositoryCustom implements JudgeRoundScoreRepositoryCustom {
>     // Spring НЕ найдет этот бин
> }
>
> // ❌ НЕПРАВИЛЬНО - другое имя
> @Repository
> public class MyCustomRepo implements JudgeRoundScoreRepositoryCustom {
>     // Spring НЕ найдет этот бин
> }
> ```
>
> **Ошибка при запуске:**
> ```
> org.springframework.beans.factory.NoSuchBeanDefinitionException:
> No qualifying bean of type 'JudgeRoundScoreRepositoryCustom' available
> ```
>
> ❌ **Проблема 2: Методы не доступны**
> ```java
> @Autowired
> private JudgeRoundScoreRepository repo;
>
> repo.getAllJudgeRoundScoresListDto(battleIds);  // ← Ошибка компиляции или рантайм исключение
> ```
>
> **Решение 1: Изменить суффикс глобально (в application.properties)**
> ```properties
> # Можно изменить дефолтный суффикс Impl на другой
> spring.data.jpa.repository.repository-impl-postfix=Custom
> ```
> Тогда класс должен называться:
> ```java
> @Repository
> public class JudgeRoundScoreRepositoryCustomCustom implements JudgeRoundScoreRepositoryCustom {
>     // Теперь Spring найдет по суффиксу Custom
> }
> ```
>
> **Решение 2: Явно указать имя бина**
> ```java
> @Repository("judgeRoundScoreRepositoryCustomImpl")  // ← Явное имя
> public class MyWeirdName implements JudgeRoundScoreRepositoryCustom {
>     // Spring найдет по явному имени
> }
> ```
>
> **Решение 3: Использовать @EnableJpaRepositories с repositoryImplementationPostfix**
> ```java
> @Configuration
> @EnableJpaRepositories(
>     basePackages = "com.example.repository",
>     repositoryImplementationPostfix = "Custom"  // ← Изменить суффикс
> )
> public class JpaConfig {
> }
> ```
>
> **Рекомендация:**
> ✅ Всегда следуйте соглашению и используйте суффикс `Impl` - это стандарт Spring Data JPA
> ✅ Не изобретайте свои схемы именования без крайней необходимости
>
> **Почему это элегантно:**
> 1. **Прозрачность:** Клиент работает с единым интерфейсом `JudgeRoundScoreRepository`
> 2. **Гибкость:** Можно комбинировать простые JPA методы и сложные jOOQ/Criteria API запросы
> 3. **Типобезопасность:** IDE видит все методы (и JPA, и кастомные)
> 4. **Maintainability:** Кастомная логика изолирована в отдельном классе
>
> **Пример с jOOQ из реального проекта:**
> ```java
> // Интерфейс
> public interface JudgeRoundScoreRepositoryCustom {
>     List<RoundScoresDto> getAllJudgeRoundScoresListDto(Set<Long> battleIds);
> }
>
> // Реализация с jOOQ
> @Repository
> @RequiredArgsConstructor
> public class JudgeRoundScoreRepositoryCustomImpl implements JudgeRoundScoreRepositoryCustom {
>     private final DSLContext dsl;
>
>     @Override
>     public List<RoundScoresDto> getAllJudgeRoundScoresListDto(Set<Long> battleIds) {
>         return dsl
>             .select(JUDGES_ROUND_SCORES.ID, JUDGES.FULLNAME, ...)
>             .from(JUDGES_ROUND_SCORES)
>             .join(JUDGES).on(JUDGES_ROUND_SCORES.JUDGE_ID.eq(JUDGES.ID))
>             .where(JUDGES_ROUND_SCORES.BATTLE_ID.in(battleIds))
>             .fetchInto(RoundScoresDto.class);
>     }
> }
>
> // Основной репозиторий
> public interface JudgeRoundScoreRepository
>     extends JpaRepository<JudgeRoundScore, Long>,
>             JudgeRoundScoreRepositoryCustom {
>
>     // JPA методы
>     boolean isJudgeScoreNotExists(long battleId, int judgeId);
>
>     // Кастомные методы (jOOQ) доступны автоматически через наследование!
>     // List<RoundScoresDto> getAllJudgeRoundScoresListDto(Set<Long> battleIds);
> }
> ```
>
> **Когда использовать этот паттерн:**
> - ✅ Нужны сложные запросы с jOOQ/Criteria API
> - ✅ Хотите избежать дублирования JPQL конструкторов DTO
> - ✅ Требуется динамическое построение запросов
> - ✅ Нужна полная типобезопасность
> - ✅ Хотите переиспользовать общую SELECT проекцию

>[!question]- Как Spring создает единый прокси-объект из JpaRepository + Custom интерфейса?
> **Вопрос:** JudgeRoundScoreRepository расширяет интерфейс JudgeRoundScoreRepositoryCustom, который реализуется классом JudgeRoundScoreRepositoryCustomImpl. При компиляции из всего этого образуется один прокси-объект?
>
> **Ответ: ДА!** Spring Data JPA создает **один динамический прокси-объект**, который объединяет все три компонента.
>
> **Структура:**
> ```java
> // 1. Основной репозиторий
> public interface JudgeRoundScoreRepository
>     extends JpaRepository<JudgesRoundScore, Long>,
>             JudgeRoundScoreRepositoryCustom {
>     // Методы Spring Data JPA (автогенерация)
>     List<JudgesRoundScore> findByBattleId(Long battleId);
> }
>
> // 2. Кастомный интерфейс
> public interface JudgeRoundScoreRepositoryCustom {
>     List<RoundScoresDto> getAllJudgeRoundScoresListDto(Set<Long> battleIds);
> }
>
> // 3. Реализация кастомных методов
> @Repository
> public class JudgeRoundScoreRepositoryCustomImpl
>     implements JudgeRoundScoreRepositoryCustom {
>
>     @Autowired
>     private DSLContext dsl;
>
>     @Override
>     public List<RoundScoresDto> getAllJudgeRoundScoresListDto(Set<Long> battleIds) {
>         // jOOQ запрос
>     }
> }
> ```
>
> **Что происходит при запуске приложения:**
>
> **Spring создает ОДИН динамический прокси-объект**, который комбинирует:
> - Методы из `JpaRepository` (автогенерируемые Spring Data)
> - Методы из `JudgeRoundScoreRepositoryCustom` (ваша реализация)
>
> **Визуализация прокси:**
> ```
> ┌─────────────────────────────────────────────────────┐
> │  Прокси-объект JudgeRoundScoreRepository           │
> │                                                      │
> │  ┌────────────────────────────────────────────┐    │
> │  │ Методы JpaRepository (Spring Data JPA)     │    │
> │  │ - findById(), save(), delete()             │    │
> │  │ - findByBattleId() (Query Methods)         │    │
> │  └────────────────────────────────────────────┘    │
> │                                                      │
> │  ┌────────────────────────────────────────────┐    │
> │  │ Методы JudgeRoundScoreRepositoryCustom     │    │
> │  │ (делегирование в CustomImpl)               │    │
> │  │ - getAllJudgeRoundScoresListDto()          │────┼──> JudgeRoundScoreRepositoryCustomImpl
> │  └────────────────────────────────────────────┘    │
> └─────────────────────────────────────────────────────┘
> ```
>
> **Как это работает под капотом:**
> ```java
> // Когда вы инжектите репозиторий:
> @Autowired
> private JudgeRoundScoreRepository repository;
>
> // Spring создает прокси примерно так (упрощенно):
> class JudgeRoundScoreRepositoryProxy implements JudgeRoundScoreRepository {
>
>     private SimpleJpaRepository<JudgesRoundScore, Long> jpaImpl; // Для JPA методов
>     private JudgeRoundScoreRepositoryCustomImpl customImpl;       // Для кастомных методов
>
>     // JPA методы делегируются в SimpleJpaRepository
>     @Override
>     public Optional<JudgesRoundScore> findById(Long id) {
>         return jpaImpl.findById(id);
>     }
>
>     // Кастомные методы делегируются в CustomImpl
>     @Override
>     public List<RoundScoresDto> getAllJudgeRoundScoresListDto(Set<Long> battleIds) {
>         return customImpl.getAllJudgeRoundScoresListDto(battleIds);
>     }
> }
> ```
>
> **Технические детали:**
>
> **1. JDK Dynamic Proxy:**
> Spring использует `java.lang.reflect.Proxy` для создания прокси-объекта во время выполнения (runtime).
>
> **2. Naming Convention:**
> Spring автоматически находит `JudgeRoundScoreRepositoryCustomImpl` по суффиксу `Impl` и связывает его с интерфейсом.
>
> **3. Один бин в контексте:**
> В Spring контексте регистрируется только **ОДИН бин** типа `JudgeRoundScoreRepository`, который является этим прокси.
>
> **Проверка в коде:**
> ```java
> @Autowired
> private JudgeRoundScoreRepository repository;
>
> public void checkProxy() {
>     System.out.println(repository.getClass().getName());
>     // Вывод: com.sun.proxy.$Proxy123 (или что-то подобное)
>     // Это и есть прокси-объект!
>
>     // Оба типа методов работают через один объект:
>     repository.findById(1L);                          // JPA метод
>     repository.getAllJudgeRoundScoresListDto(Set.of(1L)); // Кастомный метод
> }
> ```
>
> **Паттерн Composite (Композитный шаблон проектирования):**
> Spring использует паттерн Composite для объединения разных реализаций в единый интерфейс.
>
> **Итого:**
> - ✅ **Создается ОДИН прокси-объект**
> - ✅ Он объединяет методы JpaRepository и ваши кастомные методы
> - ✅ Прокси делегирует вызовы в правильные реализации
> - ✅ Для вас это выглядит как один репозиторий со всеми методами
> - ✅ Это паттерн Composite - объединение нескольких компонентов в единый интерфейс
>
> **Преимущества:**
> - Один интерфейс для всех операций (простых JPA и сложных jOOQ/Criteria API)
> - Типобезопасность - IDE видит все методы
> - Прозрачность для клиентского кода
> - Легко добавлять новые кастомные методы
>
> **Что происходит, если назвать Impl класс по-другому:**
> ```
> org.springframework.beans.factory.NoSuchBeanDefinitionException:
> No qualifying bean of type 'JudgeRoundScoreRepositoryCustom' available
> ```
> Spring не найдет реализацию и методы не будут доступны в прокси!

>[!question]- Типы прокси-объектов: статический vs динамический (JDK vs CGLIB)
>
> ## Типы прокси-объектов
>
> ### 1. Статический прокси (Static Proxy)
> Создается **вручную разработчиком** на этапе написания кода.
>
> **Пример:**
> ```java
> // Интерфейс
> public interface UserService {
>     void saveUser(String name);
> }
>
> // Реальная реализация
> public class UserServiceImpl implements UserService {
>     @Override
>     public void saveUser(String name) {
>         System.out.println("Saving user: " + name);
>     }
> }
>
> // СТАТИЧЕСКИЙ ПРОКСИ - написан вручную!
> public class UserServiceProxy implements UserService {
>     private UserService target;
>
>     public UserServiceProxy(UserService target) {
>         this.target = target;
>     }
>
>     @Override
>     public void saveUser(String name) {
>         System.out.println("BEFORE: Logging...");
>         target.saveUser(name);  // Делегирование к реальному объекту
>         System.out.println("AFTER: User saved");
>     }
> }
>
> // Использование:
> UserService service = new UserServiceProxy(new UserServiceImpl());
> service.saveUser("Igor");
> // Вывод:
> // BEFORE: Logging...
> // Saving user: Igor
> // AFTER: User saved
> ```
>
> **Проблемы статического прокси:**
> - ❌ Нужно вручную писать класс-прокси для каждого интерфейса
> - ❌ Дублирование кода (если 100 сервисов - нужно 100 прокси)
> - ❌ Сложно поддерживать (изменение интерфейса → изменение прокси)
>
> ### 2. Динамический прокси (Dynamic Proxy)
> Создается **автоматически в runtime** (во время выполнения программы).
>
> В Java есть **два механизма** динамических прокси:
>
> #### 2.1. JDK Dynamic Proxy (стандартный механизм Java)
> Работает **только с интерфейсами**.
>
> **Пример:**
> ```java
> import java.lang.reflect.InvocationHandler;
> import java.lang.reflect.Method;
> import java.lang.reflect.Proxy;
>
> // Интерфейс и реализация
> public interface UserService {
>     void saveUser(String name);
> }
>
> public class UserServiceImpl implements UserService {
>     @Override
>     public void saveUser(String name) {
>         System.out.println("Saving user: " + name);
>     }
> }
>
> // InvocationHandler - обработчик вызовов методов
> public class LoggingHandler implements InvocationHandler {
>     private Object target;
>
>     public LoggingHandler(Object target) {
>         this.target = target;
>     }
>
>     @Override
>     public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
>         System.out.println("BEFORE: " + method.getName());
>         Object result = method.invoke(target, args);  // Вызов реального метода
>         System.out.println("AFTER: " + method.getName());
>         return result;
>     }
> }
>
> // Создание динамического прокси:
> public class Main {
>     public static void main(String[] args) {
>         UserService realService = new UserServiceImpl();
>
>         // СОЗДАЕМ ДИНАМИЧЕСКИЙ ПРОКСИ В RUNTIME!
>         UserService proxyService = (UserService) Proxy.newProxyInstance(
>             UserService.class.getClassLoader(),
>             new Class<?>[]{UserService.class},  // Интерфейсы для прокси
>             new LoggingHandler(realService)
>         );
>
>         proxyService.saveUser("Igor");
>
>         // Проверка типа
>         System.out.println(proxyService.getClass().getName());
>         // Вывод: com.sun.proxy.$Proxy0 ← Динамический класс!
>     }
> }
> ```
>
> **Ограничение JDK Dynamic Proxy:**
> ```java
> // ❌ НЕ РАБОТАЕТ - нет интерфейса!
> public class UserServiceImpl {  // Без интерфейса
>     public void saveUser(String name) {
>         System.out.println("Saving user: " + name);
>     }
> }
>
> // ОШИБКА при попытке создать прокси:
> UserServiceImpl proxy = (UserServiceImpl) Proxy.newProxyInstance(...);
> // IllegalArgumentException: UserServiceImpl is not an interface
> ```
>
> #### 2.2. CGLIB Dynamic Proxy (библиотека)
> Работает **с классами БЕЗ интерфейсов** (создает подкласс через наследование).
>
> **Пример:**
> ```java
> import org.springframework.cglib.proxy.Enhancer;
> import org.springframework.cglib.proxy.MethodInterceptor;
> import org.springframework.cglib.proxy.MethodProxy;
>
> // Класс БЕЗ интерфейса
> public class UserService {
>     public void saveUser(String name) {
>         System.out.println("Saving user: " + name);
>     }
> }
>
> // Interceptor для CGLIB
> public class LoggingInterceptor implements MethodInterceptor {
>     @Override
>     public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
>         System.out.println("BEFORE: " + method.getName());
>         Object result = proxy.invokeSuper(obj, args);  // Вызов родительского метода
>         System.out.println("AFTER: " + method.getName());
>         return result;
>     }
> }
>
> // Создание CGLIB прокси:
> public class Main {
>     public static void main(String[] args) {
>         Enhancer enhancer = new Enhancer();
>         enhancer.setSuperclass(UserService.class);  // Наследуемся от класса!
>         enhancer.setCallback(new LoggingInterceptor());
>
>         UserService proxy = (UserService) enhancer.create();
>         proxy.saveUser("Igor");
>
>         // Проверка типа
>         System.out.println(proxy.getClass().getName());
>         // Вывод: UserService$$EnhancerByCGLIB$$12345678 ← CGLIB создал подкласс!
>     }
> }
> ```
>
> **Как работает CGLIB:**
> ```
>            UserService (оригинальный класс)
>                     ↑
>                     | наследование
>                     |
>     UserService$$EnhancerByCGLIB$$12345678 (динамически созданный подкласс)
>
>     @Override
>     public void saveUser(String name) {
>         interceptor.intercept(...);  // Вызов interceptor'а
>         super.saveUser(name);         // Затем вызов оригинального метода
>     }
> ```
>
> ## Сравнение всех типов прокси
>
> | Критерий | Статический прокси | JDK Dynamic Proxy | CGLIB Dynamic Proxy |
> |----------|-------------------|-------------------|---------------------|
> | **Создание** | Вручную программистом | Автоматически в runtime | Автоматически в runtime |
> | **Требует интерфейс** | Нет | ✅ ДА (обязательно!) | ❌ НЕТ (работает с классами) |
> | **Механизм** | Обычный класс | `java.lang.reflect.Proxy` | Наследование (создает подкласс) |
> | **Производительность** | Быстрый | Средняя | Чуть медленнее (из-за наследования) |
> | **Когда создается** | Compile-time | Runtime | Runtime |
> | **Название класса** | YourProxy | com.sun.proxy.$Proxy0 | YourClass$$EnhancerByCGLIB$$... |
> | **Используется в Spring** | Редко | Если есть интерфейс | Если нет интерфейса (или @Configuration) |
>
> ## Что использует Spring?
>
> Spring **автоматически выбирает** между JDK и CGLIB прокси:
>
> ```java
> // Случай 1: Есть интерфейс → JDK Dynamic Proxy
> public interface UserService {
>     void saveUser(String name);
> }
>
> @Service
> public class UserServiceImpl implements UserService {
>     @Override
>     public void saveUser(String name) {
>         System.out.println("Saving user: " + name);
>     }
> }
>
> // Spring создаст: com.sun.proxy.$Proxy123
>
>
> // Случай 2: Нет интерфейса → CGLIB Proxy
> @Service
> public class UserService {  // БЕЗ интерфейса!
>     public void saveUser(String name) {
>         System.out.println("Saving user: " + name);
>     }
> }
>
> // Spring создаст: UserService$$EnhancerBySpringCGLIB$$12345678
>
>
> // Случай 3: @Configuration → ВСЕГДА CGLIB (даже если есть интерфейс)
> @Configuration
> public class AppConfig {
>     @Bean
>     public UserService userService() {
>         return new UserServiceImpl();
>     }
> }
>
> // Spring создаст: AppConfig$$EnhancerBySpringCGLIB$$12345678
> ```
>
> ## Spring Data JPA + Custom Repository - какой прокси?
>
> В случае с `JudgeRoundScoreRepository`:
>
> ```java
> public interface JudgeRoundScoreRepository
>     extends JpaRepository<JudgesRoundScore, Long>,
>             JudgeRoundScoreRepositoryCustom {
>     // ...
> }
> ```
>
> Spring использует **JDK Dynamic Proxy**, потому что:
> - ✅ Есть интерфейс `JudgeRoundScoreRepository`
> - ✅ Spring Data создает прокси через `java.lang.reflect.Proxy`
>
> **Проверка:**
> ```java
> @Autowired
> private JudgeRoundScoreRepository repository;
>
> public void check() {
>     System.out.println(repository.getClass().getName());
>     // Вывод: com.sun.proxy.$Proxy123 ← JDK Dynamic Proxy!
>
>     System.out.println(Proxy.isProxyClass(repository.getClass()));
>     // Вывод: true
> }
> ```
>
> ## Когда Spring использует CGLIB вместо JDK?
>
> ```java
> // 1. @Configuration класс - ВСЕГДА CGLIB
> @Configuration
> public class MyConfig {
>     @Bean
>     public UserService userService() {
>         return new UserServiceImpl();
>     }
> }
>
> // 2. @Transactional на классе без интерфейса
> @Service
> @Transactional
> public class UserService {  // Нет интерфейса
>     public void saveUser(String name) { ... }
> }
>
> // 3. Принудительно через настройку
> @Configuration
> @EnableAspectJAutoProxy(proxyTargetClass = true)  // ← Форсировать CGLIB
> public class AppConfig { }
> ```
>
> ## Итого
>
> **Статический прокси:**
> - Пишется вручную
> - Используется редко (только для учебных целей или очень специфичных случаев)
>
> **Динамический прокси:**
> - **JDK Dynamic Proxy** (стандарт Java):
>   - Работает ТОЛЬКО с интерфейсами
>   - Использует `java.lang.reflect.Proxy`
>   - Быстрее CGLIB
>   - Используется Spring по умолчанию (если есть интерфейс)
>   - Название класса: `com.sun.proxy.$ProxyXXX`
>
> - **CGLIB Dynamic Proxy** (библиотека):
>   - Работает с классами БЕЗ интерфейсов
>   - Создает подкласс через наследование
>   - Используется Spring для @Configuration и классов без интерфейсов
>   - Название класса: `YourClass$$EnhancerBySpringCGLIB$$XXX`
>
> **В случае Spring Data JPA:**
> - Spring создает **JDK Dynamic Proxy** (`com.sun.proxy.$ProxyXXX`)
> - Прокси делегирует вызовы:
>   - В `SimpleJpaRepository` для JPA методов
>   - В `JudgeRoundScoreRepositoryCustomImpl` для кастомных методов

>[!question]- Что из прокси является AOP и рефлексией?
> ## Рефлексия (Reflection)
>
> **Рефлексия** - это механизм Java, который позволяет **во время выполнения (runtime)** получать информацию о классах, методах, полях и вызывать их динамически.
>
> ### Где используется рефлексия в прокси:
>
> **JDK Dynamic Proxy - ИСПОЛЬЗУЕТ РЕФЛЕКСИЮ:**
>
> ```java
> public class LoggingHandler implements InvocationHandler {
>     private Object target;
>
>     @Override
>     public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
>         System.out.println("BEFORE: " + method.getName());
>
>         // ← ЭТО РЕФЛЕКСИЯ!
>         // method - это объект java.lang.reflect.Method
>         // invoke вызывает метод через рефлексию
>         Object result = method.invoke(target, args);
>
>         System.out.println("AFTER: " + method.getName());
>         return result;
>     }
> }
> ```
>
> **Ключевые моменты:**
> - `Method method` - получен через рефлексию
> - `method.invoke(target, args)` - вызов метода через рефлексию (медленнее обычного вызова)
> - `method.getName()` - получение имени метода через рефлексию
>
> **CGLIB - НЕ ИСПОЛЬЗУЕТ РЕФЛЕКСИЮ напрямую:**
>
> ```java
> public class LoggingInterceptor implements MethodInterceptor {
>     @Override
>     public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
>         System.out.println("BEFORE: " + method.getName());
>
>         // ← ЭТО НЕ РЕФЛЕКСИЯ!
>         // proxy.invokeSuper вызывает родительский метод напрямую (через байткод)
>         // Быстрее чем method.invoke()
>         Object result = proxy.invokeSuper(obj, args);
>
>         System.out.println("AFTER: " + method.getName());
>         return result;
>     }
> }
> ```
>
> **Ключевые моменты:**
> - CGLIB генерирует байткод (bytecode generation) во время выполнения
> - `proxy.invokeSuper()` - прямой вызов через сгенерированный байткод (БЫСТРЕЕ рефлексии)
> - Но `Method method` все равно доступен для получения метаданных (это рефлексия)
>
> ## AOP (Aspect-Oriented Programming)
>
> **AOP** - это **парадигма программирования**, которая позволяет выделить сквозную функциональность (cross-cutting concerns) в отдельные модули (аспекты).
>
> ### Все прокси - это механизмы реализации AOP!
>
> **И JDK Dynamic Proxy, и CGLIB - оба являются техниками реализации AOP в Spring.**
>
> **Пример AOP концепций:**
>
> ```java
> @Service
> public class UserService {
>
>     @Transactional  // ← AOP аспект (транзакции)
>     @Cacheable      // ← AOP аспект (кеширование)
>     public void saveUser(String name) {
>         // Бизнес-логика
>         System.out.println("Saving user: " + name);
>     }
> }
> ```
>
> **Что происходит под капотом (AOP через прокси):**
>
> ```java
> // Spring создает ПРОКСИ (через JDK или CGLIB)
> UserService proxy = createProxy(new UserServiceImpl());
>
> // Когда вы вызываете:
> proxy.saveUser("Igor");
>
> // Прокси выполняет (это и есть AOP):
> 1. BEFORE (advice): Открыть транзакцию      ← Аспект @Transactional
> 2. BEFORE (advice): Проверить кеш            ← Аспект @Cacheable
> 3. TARGET: Вызвать реальный метод saveUser() ← Бизнес-логика
> 4. AFTER (advice): Сохранить в кеш           ← Аспект @Cacheable
> 5. AFTER (advice): Закоммитить транзакцию    ← Аспект @Transactional
> ```
>
> **Терминология AOP:**
>
> ```java
> // Join Point - точка в программе, где можно применить аспект (вызов метода)
> proxy.saveUser("Igor");  // ← Join Point
>
> // Advice - дополнительная логика, которую нужно выполнить
> // Типы Advice:
> @Before           // Перед вызовом метода
> @After            // После вызова метода
> @Around           // Вокруг вызова метода (контроль ДО и ПОСЛЕ)
> @AfterReturning   // После успешного выполнения
> @AfterThrowing    // После исключения
>
> // Pointcut - выражение, которое определяет, к каким методам применять аспект
> @Pointcut("execution(* com.example.service.*.*(..))")  // Все методы в service пакете
>
> // Aspect - модуль, который объединяет Advice + Pointcut
> @Aspect
> @Component
> public class LoggingAspect {
>
>     @Around("execution(* com.example.service.*.*(..))")  // ← Pointcut
>     public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {  // ← Advice
>         long start = System.currentTimeMillis();
>
>         Object result = joinPoint.proceed();  // Вызов реального метода
>
>         long executionTime = System.currentTimeMillis() - start;
>         System.out.println(joinPoint.getSignature() + " executed in " + executionTime + "ms");
>         return result;
>     }
> }
> ```
>
> ## Визуализация: Рефлексия vs AOP
>
> ```
> ┌─────────────────────────────────────────────────────────────┐
> │                    РЕФЛЕКСИЯ (Reflection)                   │
> │  Механизм Java для работы с классами в runtime              │
> ├─────────────────────────────────────────────────────────────┤
> │  • java.lang.reflect.Method                                 │
> │  • method.invoke(target, args)                              │
> │  • Class.forName("...")                                     │
> │  • field.get(object), field.set(object, value)              │
> │                                                              │
> │  ИСПОЛЬЗУЕТСЯ В:                                            │
> │  ✅ JDK Dynamic Proxy (напрямую)                            │
> │  ⚠️ CGLIB (частично, для метаданных)                       │
> └─────────────────────────────────────────────────────────────┘
>
> ┌─────────────────────────────────────────────────────────────┐
> │              AOP (Aspect-Oriented Programming)              │
> │  Парадигма программирования для модуляризации               │
> │  сквозной функциональности                                  │
> ├─────────────────────────────────────────────────────────────┤
> │  Концепции AOP:                                             │
> │  • Aspect (аспект)                                          │
> │  • Advice (совет: @Before, @After, @Around)                 │
> │  • Join Point (точка соединения)                            │
> │  • Pointcut (точка среза)                                   │
> │                                                              │
> │  РЕАЛИЗУЕТСЯ ЧЕРЕЗ:                                         │
> │  ✅ JDK Dynamic Proxy (для интерфейсов)                     │
> │  ✅ CGLIB Proxy (для классов без интерфейсов)               │
> │  ✅ AspectJ (compile-time weaving)                          │
> └─────────────────────────────────────────────────────────────┘
> ```
>
> ## Примеры в Spring
>
> ### 1. Spring AOP через JDK Dynamic Proxy
>
> ```java
> // Интерфейс
> public interface PaymentService {
>     void processPayment(double amount);
> }
>
> // Реализация
> @Service
> public class PaymentServiceImpl implements PaymentService {
>
>     @Transactional  // ← AOP аспект
>     @Override
>     public void processPayment(double amount) {
>         System.out.println("Processing payment: " + amount);
>     }
> }
>
> // Spring создаст JDK Dynamic Proxy:
> // com.sun.proxy.$Proxy123 implements PaymentService
>
> // Прокси перехватит вызов и добавит:
> // 1. Открытие транзакции (BEFORE)
> // 2. Вызов метода через рефлексию
> // 3. Коммит транзакции (AFTER)
> ```
>
> ### 2. Spring AOP через CGLIB
>
> ```java
> // Класс БЕЗ интерфейса
> @Service
> public class PaymentService {  // Нет интерфейса!
>
>     @Transactional  // ← AOP аспект
>     public void processPayment(double amount) {
>         System.out.println("Processing payment: " + amount);
>     }
> }
>
> // Spring создаст CGLIB Proxy:
> // PaymentService$$EnhancerBySpringCGLIB$$12345678 extends PaymentService
>
> // Прокси перехватит вызов и добавит:
> // 1. Открытие транзакции (BEFORE)
> // 2. Вызов родительского метода через байткод (НЕ рефлексия!)
> // 3. Коммит транзакции (AFTER)
> ```
>
> ### 3. Кастомный AOP аспект
>
> ```java
> @Aspect
> @Component
> public class PerformanceAspect {
>
>     // Pointcut - где применять
>     @Pointcut("execution(* com.example.service.*.*(..))")
>     public void serviceMethods() {}
>
>     // Advice - что делать
>     @Around("serviceMethods()")  // ← Это AOP!
>     public Object measureExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
>         long start = System.currentTimeMillis();
>
>         // Вызов реального метода
>         Object result = joinPoint.proceed();
>
>         long time = System.currentTimeMillis() - start;
>         System.out.println(joinPoint.getSignature() + " took " + time + "ms");
>
>         return result;
>     }
> }
> ```
>
> ## Итого
>
> | Концепция | Что это | Где используется в прокси |
> |-----------|---------|---------------------------|
> | **Рефлексия** | Механизм Java для работы с классами в runtime | ✅ JDK Dynamic Proxy (напрямую через `method.invoke()`)<br>⚠️ CGLIB (частично, для метаданных) |
> | **AOP** | Парадигма программирования для модуляризации сквозной функциональности | ✅ JDK Dynamic Proxy (механизм реализации)<br>✅ CGLIB (механизм реализации)<br>✅ AspectJ (compile-time weaving) |
>
> **Прокси - это инструмент для реализации AOP:**
> - `@Transactional` - AOP через прокси
> - `@Cacheable` - AOP через прокси
> - `@Async` - AOP через прокси
> - Spring Security - AOP через прокси
>
> **Рефлексия - это механизм вызова методов в JDK Dynamic Proxy:**
> - `method.invoke()` - рефлексия (медленнее)
> - `proxy.invokeSuper()` в CGLIB - НЕ рефлексия (быстрее)

>[!question]- **AOP** - это
> **парадигма программирования**, которая позволяет выделить сквозную функциональность (cross-cutting concerns) в отдельные модули (аспекты).
> «Сквозная функциональность» (cross‑cutting concern) — это такая часть поведения программы, которая нужна во многих местах и слоях приложения сразу, а не живёт в одном чётко выделенном модуле или классе
> Типичные примеры сквозной функциональности: логирование, аутентификация и проверка прав доступа, обработка исключений, транзакции, кеширование. Их код «размазывается» по контроллерам, сервисам, репозиториям и т.д., из‑за чего он дублируется и переплетается с бизнес‑логикой. АОП как раз позволяет собрать такой повторяющийся «размазанный» код в отдельные аспекты и подключать его декларативно в нужных точках выполнения.[](http://www.finecosoft.ru/spring-aop) 



Два механизма в Spring/Hibernate                                                                                                                                                                                                                                                                                  
   
  Spring AOP (прокси) — обрабатывает Spring:                                                                                                                                                                                                                                                                        
  - @Transactional
  - @Cacheable
  - @Async
  - @PreAuthorize

  Создаёт прокси-обёртку вокруг бина, перехватывает вызовы методов.

  Hibernate маппинг — обрабатывает Hibernate напрямую:
  - @Convert
  - @Enumerated
  - @Column
  - @OneToMany

  Никакого прокси — просто метаданные для ORM при работе с JDBC.

  ---
  Как понять кто обрабатывает?

  Простой ориентир:

  ┌──────────────────────────────────────────┬─────────────────────────┐
  │                  Вопрос                  │          Ответ          │
  ├──────────────────────────────────────────┼─────────────────────────┤
  │ Аннотация на методе/классе бина?         │ скорее всего Spring AOP │
  ├──────────────────────────────────────────┼─────────────────────────┤
  │ Аннотация на поле Entity?                │ скорее всего Hibernate  │
  ├──────────────────────────────────────────┼─────────────────────────┤
  │ Аннотация из пакета org.springframework? │ Spring                  │
  ├──────────────────────────────────────────┼─────────────────────────┤
  │ Аннотация из пакета jakarta.persistence? │ JPA/Hibernate           │
  └──────────────────────────────────────────┴─────────────────────────┘

  ---
  Но это именно ориентир, не правило — например @Validated из Spring стоит на классе, но работает через AOP, а @Column стоит на поле и к прокси вообще не относится.


Но если честно — это скорее вопрос определения, чем абсолютная истина:

  Если конвертер применяется к каждому LocalDateTime-полю во всех сущностях — это вполне можно назвать сквозным поведением на уровне данных. JPA @Converter(autoApply = true) по своей механике очень похож на аспект — декларативно подключается везде, где нужный тип.

  Разница скорее в намерении:
  - АОП — добавляет поведение (логирование, транзакции)
  - Конвертер — меняет представление данных

  Но это философское разграничение, а не техническое. Ты прав, что механически это работает аналогично.


# [[Spring cloud]] 

# [[IoC Spring]]

# [[Spring тесты]] 

# [[Spring Kafka]] 

