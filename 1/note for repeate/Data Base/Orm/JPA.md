
1. [ ] проверь **` LEFT JOIN FETCH`** по числу подзапросов - сравни с LEFT JOIN
2. [ ] Cash first level and other 
# База 
>[!question]- что это 
>JPA - спецификация ORM. Состоит из аннотаций, интерфейсов и контрактов EntityManager - кода который ходит в БД нет.

>[!question]- Hibernate  это
>одна из реализаций JPA. У Hibernate так же есть свое родное API (HQL, Session, Criteria
>

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
> см. [[application.properties#^jpa-stat-log]]
# Оптимизация запросов 

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

# Подводные камни
[[JPA equals hashCode]] 

# Подходы к работе 
>[!question]- Можно ли использовать @Entity как класс с полями , а действия над ними (бизнес логику) проводить в другом классе который от него наследуется - нормальный ли это подход ? Или обычно методы бизнес логики находятся в том же @Entity  классе ?
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