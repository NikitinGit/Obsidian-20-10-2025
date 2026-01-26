# Задачи
1. [x] что можно сделать на спринг и что делают (мобильные приложения, десктопные приложения, веб приложения ... )
2. [ ] благодаря поддержке транзакций, интеграции с JMS, Kafka и другими сообщениями, Spring активно используется для создания банковских систем, страхования, складских систем и документооборота.
3. [x] напиши прмер приложения с веб запросами , без @SpringBootApplication 
4. [x] может ли @Configuration класс просканировать бины в других пакетах с помощью @ComponentScan - да
5. [x] в каком порядке инициализируются бины - ДОПИШИ здесь и пример  с инициализацией и логами  18.05 - 19.41
6. [x] чем отличается @Bean  созданный внутри @Configuration класса от просто внутри класса ?- ДОПИШИ пример из чатджпт 
7. [x] что такое CGLIB 
8. [x] Можно ли использовать @Entity как класс с полями , а действия над ними (бизнес логику) проводить в другом классе который от него наследуется - нормальный ли это подход ? Или обычно методы бизнес логики назодятся в том же @Entity  классе ? - нет
9. [ ] Попробуй jOOQ https://www.jooq.org/ 
10. [ ] Когда происходит не явный inner join при выборке - допиши возникает ли если c.country это не сущность а int 
11. [ ] Проблемы с Hibernate прокси:- как зайбернейт создает проки объект в которм equals | hashcode не работает - 
12. [ ] Безопасно ли при выборке делать сравнение с объектом сущостью или лучше с id - List\<JudgeScore> findAllByBattleAndJudge(Battle battle, Judge judge); \-  когда могут возникнуть проблеммы ?
13. [ ]  Почему в JudgeScoreRepository можно делать выборку других сущностей не относящихся к этому репозиторию и правильно ли так делать 
14. [ ] Прочитай https://docs.spring.io/spring-framework/reference/web.html и сравни со своим проектом
15. [x] Попробуй вариант Решение 2: QueryDSL (IDE видит usage!) в клоде - который должен показывать usage  
16. [ ] как кэшировать гет запросы 
17. [ ] Domain-Driven Design (Доменно-ориентированное проектирование)  
18. [ ] Всегда ли есть смысл делать связь на уровне ентити а не только на уровне БД ?
19. [ ] какие аннотации надо знать на собеседовании 
20. [ ] @Profile
21. [ ] @ConditionProperty
22. [ ] КЭШ первого и второго уровня
23. [ ] возможно ли подлкючиться к бд без application.properties
24. [ ] как запустить не ddl миграции в спринг - посмотри в MigrationService.java 
25. [ ] встроенный серверы в Spring boot - Tomcat, Jetty 
26. [ ] отличие @GetMapping("/hello") от @RequestMapping(value = "/hello", method = RequestMethod.GET)
27. [ ] AOP в Spring 
28. [ ] как создается изолирванный конитекст 
29. [ ] hikary poll 
30. [ ] Подключения к БД:  credentials, pool settings
31. [ ] елк стек, актуатор , графана 

>[!question]- Когда происходит не явный inner join при выборке 
>```
>@Query("SELECT new com.strikerstat.webapp.dto.olympic_events.CityDto(c.country, c.region) "
>```
> и country - это объект класса Country, если он null - то в  выборку не попадает

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


