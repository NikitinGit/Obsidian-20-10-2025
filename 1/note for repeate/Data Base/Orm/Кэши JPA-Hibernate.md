# Кэши JPA/Hibernate — L1, L2, Query Cache

>[!question]- Где искать L1/L2/Query Cache — вынесено в отдельную памятку
> Единственное, что важно знать именно в контексте прокси: L1 (Persistence Context) — это то, что определяет, получишь ли ты **прокси или реальный объект** при обращении к entity (см. разделы выше "Прокси в Hibernate" — вся механика `clear()`/`detach()`/lazy-инициализации завязана именно на L1).

>[!question]- Сколько кэшей и как называются — сводка перед деталями
> Три штуки, которые часто путают:
>
> | Кэш | Другие названия | JPA-спека или Hibernate? |
> |---|---|---|
> | **L1 (first-level cache)** | Persistence Context, иногда "Session cache" | **JPA-спека**, обязательно — стандартизовано, одинаково у всех провайдеров, отключить нельзя |
> | **L2 (second-level cache)** | Shared cache, SessionFactory-level cache | **Наполовину** — JPA (с версии 2.0) стандартизует только точки управления (`@Cacheable`, `SharedCacheMode`, `EntityManagerFactory.getCache()`), а сам механизм (concurrency-стратегии, движок кэша) — целиком Hibernate |
> | **Query Cache** | — | **Только Hibernate**, в спеке JPA вообще не описан — кэширует не сущности, а результаты конкретных JPQL/HQL-запросов (список id), требует включённого L2 |

>[!question]- Это то же самое, что Spring Cache (`@Cacheable`/`CacheManager`)?
> **Нет, два независимых механизма**, разного уровня:
>
> - **Spring Cache abstraction** (`@Cacheable`/`@CacheEvict`/`CacheManager`) — кэширует **возвращаемое значение метода** по его аргументам. Работает через **Spring AOP**, тот же механизм прокси, что и `@Transactional` (см. [[Proxy Object]], [[Spring AOP]]): прокси перехватывает вызов метода и проверяет кэш ДО вызова. Технологически-агностичен (Redis/Ehcache/Caffeine/in-memory), метод может вообще не иметь отношения к БД. Принадлежит **Spring Framework**.
> - **L1/L2/Query Cache** — кэшируют **состояние Entity/строки БД** внутри персистенс-слоя, работают **внутри `EntityManager`/`Session`**, без всякого AOP-перехвата на уровне сервисного метода. Принадлежат **JPA-спеке/Hibernate**.
>
> Они могут стоять друг над другом в одном приложении (`@Cacheable` на методе репозитория поверх Hibernate с включённым L2 — данные кэшируются дважды, на двух независимых уровнях), но это не варианты одного и того же механизма, а два разных владельца, решающих одну прикладную задачу на разных этажах.

>[!question]- Какие уровни кэша есть в Hibernate
> Три понятия, которые часто путают:
> 1. **L1 cache** = **First-Level Cache** = **Persistence Context** — встроенный, обязательный, на одну сессию. Хранит managed-сущности по `(EntityClass, id)`.
> 2. **L2 cache** = **Second-Level Cache** — опциональный, общий на всю `SessionFactory` (т. е. на всё приложение). Хранит entity между сессиями. Включается явно, нужен сторонний провайдер.
> 3. **Query Cache** — отдельный опциональный кэш, который кэширует **результаты JPQL/criteria-запросов** (список id'ов). Часто путают с L2, но это **другая** штука, включается отдельно.
> Spring `@Cacheable` (на уровне приложения, Redis/Caffeine на бизнес-методах) — это **не Hibernate**, это уровень выше. Хибернейтовские кэши о нём ничего не знают.

>[!question]- L1 cache (Persistence Context) — кратко главное
> - **Scope**: одна сессия / один EntityManager / одна транзакция (по умолчанию).
> - **Включён всегда**, отключить нельзя.
> - **Что хранит**: managed-сущности по ключу `(EntityClass, id)`.
> - **Кто сюда лезит**: `find`, `findById`, `getReference`, lazy-связи; JPQL/native сверяет id'ы из результата с PC.
> - **Главная гарантия**: identity map (одна строка БД ⇒ одна Java-ссылка) внутри сессии.
> - **Когда очищается**: `entityManager.clear()`, `detach()`, конец `@Transactional`-метода.
> Подробности про то, когда PC определяет "прокси или реальный объект" — см. [[Proxy Object]], раздел "Прокси в Hibernate".

>[!question]- Что такое L2 cache и чем отличается от L1
> **L2 cache (Second-Level Cache)** — общий кэш на всю `SessionFactory`, живёт между сессиями. Когда `find(Event.class, 5)` промахивается мимо PC (L1), Hibernate проверяет **L2** перед тем, как идти в БД. Если в L2 есть entity с этим id — возвращает его (не сам Java-объект, а копию состояния, чтобы не делиться между сессиями).
> Ключевые отличия от L1:
> | Аспект | L1 (PC) | L2 |
> |---|---|---|
> | Scope | Одна сессия | Вся SessionFactory (всё приложение) |
> | Включён | Всегда | По умолчанию выключен, нужно явно включить |
> | Что хранит | Managed-объекты (Java-ссылки) | Сериализованное состояние entity (id + поля) |
> | Провайдер | Сам Hibernate | Внешний: Ehcache, Caffeine, Hazelcast, Infinispan |
> | Гарантия identity map | Да (одна ссылка в сессии) | Нет — каждая сессия получает свою копию |
> | Кэшируются связи | Нет (есть managed-объект, ссылки живые) | Только если включить collection cache отдельно |

>[!question]- Как включить L2 cache
> Три шага:
> 1. Подключить провайдер — например, Ehcache:
>    ```xml
>    <dependency>
>      <groupId>org.hibernate.orm</groupId>
>      <artifactId>hibernate-jcache</artifactId>
>    </dependency>
>    <dependency>
>      <groupId>org.ehcache</groupId>
>      <artifactId>ehcache</artifactId>
>    </dependency>
>    ```
> 2. В `application.properties`:
>    ```properties
>    spring.jpa.properties.hibernate.cache.use_second_level_cache=true
>    spring.jpa.properties.hibernate.cache.region.factory_class=jcache
>    spring.jpa.properties.hibernate.javax.cache.provider=org.ehcache.jsr107.EhcacheCachingProvider
>    ```
> 3. Аннотировать entity, которую хотим кэшировать:
>    ```java
>    @Entity
>    @Cacheable
>    @Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
>    public class Country { ... }
>    ```
> Без аннотации `@Cacheable` + `@Cache` entity в L2 НЕ попадёт, даже если cache включён глобально.

>[!question]- Какие 4 стратегии конкурентности у L2 (`CacheConcurrencyStrategy`)
> | Стратегия | Когда применять |
> |---|---|
> | `READ_ONLY` | Данные **никогда не меняются** после загрузки (справочники: страны, валюты). Самый быстрый, без блокировок. При попытке UPDATE — exception. |
> | `NONSTRICT_READ_WRITE` | Меняются редко, можно жить с краткой неконсистентностью (несколько мс) — например, теги, категории. Без soft locks. |
> | `READ_WRITE` | Полная консистентность через soft locks: при UPDATE entity помечается "грязной", параллельные читатели идут в БД. Подходит для большинства изменяемых данных. |
> | `TRANSACTIONAL` | Полная транзакционность через JTA. Нужен распределённый кэш с поддержкой XA (Infinispan, Hazelcast). Тяжёлая и редко используемая опция. |
> Если не знаете — `READ_WRITE`. Для справочников — `READ_ONLY`.

>[!question]- Когда L2 cache СРАБАТЫВАЕТ, а когда промахивается — главная грабля
> L2 срабатывает **только при загрузке по id**:
> - `entityManager.find(Class, id)` ✓
> - `repository.findById(id)` ✓
> - `entityManager.getReference(Class, id)` ✓
> - Lazy-связь `parent.getChild()` (внутри — find по id) ✓
> - `repository.findByName("foo")` ✗ — это JPQL под капотом, идёт в БД
> - `repository.findAll()` ✗ — тоже JPQL
> - `@Query("SELECT ...")` любой ✗ — JPQL/native всегда в БД
> - Criteria/Specifications ✗
> Чтобы кэшировать **результаты запросов** — нужен **Query Cache** (отдельно).
> Это самая частая разочарованность: "включил L2, а SQL'ей не уменьшилось" — потому что код использует `findByXxx`-методы, а не `findById`.

>[!question]- Что такое Query Cache и почему его включают отдельно
> **Query Cache** хранит мапу `(JPQL + параметры) → List<id>`. При повторном таком же запросе Hibernate берёт id'ы из кэша, потом загружает entity (из L2 или БД). Это **другой** кэш, чем L2 entity cache.
> Включение:
> ```properties
> spring.jpa.properties.hibernate.cache.use_query_cache=true
> ```
> На уровне запроса:
> ```java
> @QueryHints(@QueryHint(name = "org.hibernate.cacheable", value = "true"))
> @Query("SELECT e FROM Event e WHERE e.name = :name")
> List<Event> findByNameCached(@Param("name") String name);
> ```
> Подводный камень: Query Cache **инвалидируется при ЛЮБОМ изменении в таблицах**, которые он трогает. Если таблица часто пишется — кэш бесполезен (промах за промахом). Используется ограниченно: для редко меняющихся справочных запросов.

>[!question]- Когда L2 cache реально полезен, а когда вреден
> **Полезен**:
> - Справочные данные (страны, валюты, типы) — много читают по id, почти не пишут.
> - "Горячие" entity, которые много раз ищутся по id в течение секунд (например, текущий пользователь).
> - DB-bottleneck: сетевой round-trip к БД дороже, чем in-memory кэш.
> **Вреден или бесполезен**:
> - Write-heavy данные: L2 чаще инвалидируется, чем используется.
> - Запросы, а не lookup'ы по id — L2 их не кэширует.
> - Multi-instance приложение **без распределённого кэша**: каждый JVM имеет свой L2 → инстанс А обновил entity, в L2 инстанса Б осталась старая копия → **stale data**. Решение: Hazelcast / Infinispan / Redis с инвалидацией, или вообще не использовать L2 в кластере.
> - Native запросы пишут в БД мимо Hibernate → L2 не знает об изменении → stale data. Нужно вручную `evict` или `@SQLInsert`/`@SQLUpdate` с пометкой регионов.

>[!question]- Главные подводные камни L2 cache
> 1. **Native query, миграция, ручной SQL** меняют БД, но L2 об этом не знает → выдаёт устаревшие данные. Решения: `sessionFactory.getCache().evict(...)`, `@QueryHint(HINT_FLUSH_MODE)`, или не использовать L2 для таких сценариев.
> 2. **Кластеризация**: дефолтный provider (Ehcache local) держит кэш в JVM → в кластере из 3 нод у каждой свой кэш, изменения не видны соседям. Нужен distributed cache.
> 3. **L2 не кэширует ассоциации** автоматически: `Event.getBids()` пойдёт в БД, даже если сам Event из L2. Нужно `@Cache` на коллекции отдельно.
> 4. **Размер региона / eviction**: без настройки кэш может расти до OOM. Указывают `maxEntries`, `timeToLiveSeconds`, `timeToIdleSeconds`.
> 5. **Тесты**: L2 включённый в тестах ломает изоляцию между @Test-методами. Обычно отключают в тестовом профиле.

>[!question]- Сводная таблица: что куда смотрит при загрузке entity
> Загрузка `find(Event.class, 5)` с включенным L2:
> ```
> 1. L1 (PC текущей сессии): есть? → вернуть managed-объект.       ← всегда работает
> 2. L2 (общий между сессиями): есть? → собрать entity из L2,
>    положить в L1, вернуть.                                       ← если включен и аннотирован
> 3. БД: SELECT, собрать entity, положить в L1 и L2, вернуть.
> ```
> JPQL/native `@Query`:
> ```
> 1. Query Cache: есть кэш этого запроса? → взять id'ы.            ← если включён и @Cacheable
>    Иначе SQL в БД, получить id'ы.
> 2. Для каждого id: проверить L1 → L2 → БД.
> 3. Результат — список managed entity.
> ```

>[!question]- Главное правило про кэши одной фразой
> **L1 (PC) — это не кэш ради скорости, это identity map ради корректности.** Он работает сам, отключить нельзя, и забывать про него не нужно. **L2 и Query Cache — это уже оптимизация**, и они выключены по умолчанию специально: 80% проектов их не включают вообще, потому что выигрыш от уменьшения SQL'ей меньше, чем риск получить stale data в кластере. Прежде чем включать L2, проверьте, что **БД действительно узкое место**, а не код / сериализация / N+1 ([[JPA N+1 problem]]).