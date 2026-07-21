# Проблема N+1 в JPA / Hibernate
1. [ ] Если вообще не исопльзовать в @Entity связи - то проблема N+1 снимется ?
2. [ ] проверь **`LEFT JOIN FETCH`** по числу подзапросов - сравни с LEFT JOIN
3. [ ] всегда ли есть смысл делать связь на уровне Entity, а не только в БД (см. также [[Spring Base]])

>[!question]- Что такое N+1 проблема — одной фразой
> Когда вместо одного SQL'я с JOIN'ом Hibernate выполняет **1 запрос за родительскими сущностями + ещё N запросов по одному на каждый дочерний объект**. Возникает на ровном месте при обращении к lazy-связям внутри цикла.

>[!question]- Пример N+1 запросов
> ```java
> Event event = eventRepository.findEventByEventId(eventId).orElseThrow(...);   // 1 SELECT events
> List<EventBidFighter> bids = event.getEventBidFighters();                     // 1 SELECT bids (lazy collection)
> for (EventBidFighter bid : bids) {
>     bid.getFighter().getFirstName();                                          // 1 SELECT fighter — НА КАЖДЫЙ bid
> }
> ```
> - **1** запрос за event.
> - **1** запрос за коллекцией `bids` (`@OneToMany(mappedBy = "event")` — lazy).
> - **N** запросов за `Fighter`'ами: каждый `bid.getFighter()` — это lazy-прокси `@ManyToOne(fetch = LAZY)`, и при чтении любого поля проксь инициализируется отдельным SELECT'ом.
> Итого `1 + 1 + N`. При 399 bid'ах = **401 запрос**.

>[!question]- Как заметить N+1 в логах
> Включить вывод метрик и SQL'я:
> ```properties
> spring.jpa.show-sql=true
> spring.jpa.properties.hibernate.generate_statistics=true
> logging.level.org.hibernate.stat=DEBUG
> ```
> В логах появится блок `Session Metrics { ... N JDBC statements; ... }`. Если N резко больше ожидаемого (1–2) — это N+1.

>[!question]- Реальные метрики из проекта — сравнение трёх вариантов
> На одном и том же event с 399 bid'ами:
> 
> | Вариант | JDBC statements | Prep (ns) | Exec (ns) |
> |---|---:|---:|---:|
> | Native `SELECT * FROM events` + lazy в цикле | **401** | 20 691 610 | 69 240 066 |
> | JPQL `JOIN FETCH` | **1** | 5 336 679 | 40 720 417 |
> | `@EntityGraph` | **1** | 3 301 268 | 30 019 085 |
> 
> Время на исполнение упало в ~2 раза, на подготовку — в ~6 раз, но **главное — 400 лишних roundtrip'ов до БД исчезли**. На реальной сети между приложением и БД эффект ещё драматичнее (каждый roundtrip = десятки/сотни микросекунд сетевой задержки).

>[!question]- Решение №1: JPQL с `JOIN FETCH`
> ```java
> @Query("SELECT DISTINCT e FROM Event e " +
>        "LEFT JOIN FETCH e.eventBidFighters bf " +
>        "LEFT JOIN FETCH bf.fighter " +
>        "WHERE e.eventId = :eventId")
> Optional<Event> findEventWithBidsAndFighters(@Param("eventId") Integer eventId);
> ```
> Один SELECT с двумя LEFT JOIN'ами. Hibernate сразу заполняет коллекцию `eventBidFighters` и для каждого bid'а — связанного `Fighter`. Никаких lazy-инициализаций потом не будет.

>[!question]- Чем `JOIN FETCH` отличается от обычного `JOIN` в JPQL
> Разница — в слове `FETCH`: оно **жадно инициализирует** ассоциацию в граф объектов, а не просто соединяет таблицы.
> - **`JOIN c.event`** (обычный): генерит SQL `JOIN`, нужен чтобы **фильтровать/сортировать/использовать в WHERE** поля связанной таблицы. Ассоциацию НЕ инициализирует — `EventChallenge.event` остаётся **ленивым прокси**, колонки event в проекцию не попадают. Обращение к `challenge.getEvent().getX()` потом → отдельный ленивый SELECT (N+1) внутри сессии, либо **`LazyInitializationException`** вне сессии / на detached-объекте.
> - **`JOIN FETCH c.event`**: тот же SQL `JOIN`, **плюс** колонки event тянутся в этот же запрос и `EventChallenge.event` заполняется **реальной инициализированной сущностью** (не прокси). После этого доступ к полям event работает без доп. запросов и без LazyInit, даже на detached.
> 
> Формула: `JOIN` = «соедини для условий запроса, ассоциация остаётся ленивой»; `JOIN FETCH` = «соедини И загрузи ассоциацию в объект сразу».
> 
> **`LEFT` vs внутренний при FETCH:** `JOIN FETCH` (inner) **выкидывает** строки, где ассоциация `null`; `LEFT JOIN FETCH` их сохраняет (ассоциация останется `null`). Поэтому `LEFT JOIN FETCH` берут для **nullable**-связей: на `@ManyToOne optional=false` / NOT NULL FK → `JOIN FETCH`, на nullable → `LEFT JOIN FETCH`.
> 
> **Когда критично:** если entity обрабатывается **вне транзакции** (detached — например, фоновый воркер прочитал пачку и закрыл сессию), то всё, к чему полезешь через геттеры, обязано быть подтянуто через `JOIN FETCH` заранее, иначе `LazyInitializationException`. Это основной способ контролировать загрузку точечно вместо EAGER — см. [[JPA N+1 problem]].

>[!question]- Зачем `DISTINCT` в JPQL с JOIN FETCH коллекции
> JOIN с коллекцией размножает строки родительской сущности: если у event'а 399 bid'ов, SQL вернёт 399 строк, каждая с тем же event'ом. Без `DISTINCT` Hibernate соберёт 399 ссылок на один и тот же Java-объект в результат.
> `DISTINCT` в JPQL — это **подсказка Hibernate**, чтобы он отсёк дубли root-сущности на уровне маппинга, а не передавал `DISTINCT` в SQL (для большой выборки `SELECT DISTINCT` в SQL'е — лишняя нагрузка на БД). В Hibernate 6 это поведение по умолчанию (через `PassDistinctThrough = false`), в более ранних версиях управлялось `QueryHint`.

>[!question]- Решение №2: декларативно через `@EntityGraph`
> ```java
> @EntityGraph(attributePaths = {"eventBidFighters", "eventBidFighters.fighter"})
> @Query("SELECT e FROM Event e WHERE e.eventId = :eventId")
> Optional<Event> findEventByEventIdGraph(@Param("eventId") Integer eventId);
> ```
> Эффект тот же — один SELECT с JOIN'ами. Spring Data сам строит JOIN FETCH по перечисленным путям, JPQL остаётся простой.
> Удобно, когда **разные методы** хотят разные комбинации связей: одну и ту же JPQL переиспользуют, а fetch-плана задают разные через аннотацию.

>[!question]- Решение №3: `@BatchSize` — компромисс
> На классе `Fighter` или на коллекции:
> ```java
> @BatchSize(size = 50)
> @ManyToOne(fetch = FetchType.LAZY)
> private Fighter fighter;
> ```
> Hibernate всё равно загружает лениво, но **пачками по N через `WHERE id IN (?, ?, ...)`**. 399 запросов превратятся в ~8.
> Когда полезно: когда не хочется тащить связи всегда (как сделал бы JOIN FETCH), но если уж тащим — то пакетно.

>[!question]- Почему `nativeQuery = true` не спасает от N+1
> ```java
> @Query(value = "SELECT * FROM `events` WHERE event_id = :eventId", nativeQuery = true)
> ```
> Native-запрос тащит **только колонки таблицы `events`**. Никакие `JOIN FETCH` в нём не работают (native ≠ JPQL). Связанные коллекции и `@ManyToOne` остаются lazy и догружаются по факту обращения = N+1 в полный рост.

>[!question]- Можно ли вместо JOIN FETCH сделать `@ManyToOne(fetch = EAGER)`
> Можно, но **нельзя** — почти всегда плохая идея:
> - EAGER тащит связь **в каждом** запросе, который касается родителя, даже когда она не нужна.
> - С коллекциями EAGER (`@OneToMany(fetch = EAGER)`) при пагинации даёт `HHH000104: firstResult/maxResults specified with collection fetch; applying in memory` — Hibernate грузит ВСЁ в память и режет постранично там.
> - Связи в Hibernate должны быть LAZY по умолчанию, а fetch-стратегия — задаваться **на конкретный запрос** через JOIN FETCH/EntityGraph.

>[!question]- Что такое `MultipleBagFetchException` и как его обойти
> При попытке `LEFT JOIN FETCH` сразу к **двум коллекциям типа `List`** (`Bag` в терминах Hibernate) бросается:
> ```
> cannot simultaneously fetch multiple bags
> ```
> Причина: декартово произведение двух коллекций даст M×N строк, и Hibernate не может корректно дедуплицировать.
> Решения:
> - Заменить `List` на `Set` хотя бы у одной из коллекций — `Set` (Hibernate `PersistentSet`) дедуплицируется на уровне маппинга, можно фетчить несколько.
> - Разбить на два запроса: первый фетчит первую коллекцию, второй — вторую. Если оба запроса вернут один и тот же root-entity в рамках одной сессии (identity map), вторая коллекция привяжется к нему же.

>[!question]- Что значит warning `HHH000104: firstResult/maxResults specified with collection fetch`
> Hibernate говорит: "ты попросил `JOIN FETCH` коллекции и одновременно `setFirstResult/setMaxResults` (пагинацию). Сделать это правильно на стороне БД невозможно (LIMIT отрежет строки JOIN'а, а не root-сущности), поэтому я загружаю ВСЮ выборку в память и режу постранично там". Это бомба: на больших данных уронит JVM.
> Решения:
> - Сделать **два запроса**: первый — пагинированный SELECT только id'ов (`SELECT e.id FROM Event e ... LIMIT`); второй — `SELECT ... FROM Event e LEFT JOIN FETCH ... WHERE e.id IN (:ids)`.
> - Или фетчить без пагинации.
> - Или использовать `@BatchSize` вместо `JOIN FETCH`.

>[!question]- Когда лучше DTO-проекция вместо `JOIN FETCH`
> Если вам нужны только некоторые поля (например, для списка / экспорта в CSV), а не полноценные managed-сущности — проекция в DTO часто чище:
> ```java
> @Query("SELECT new com.example.FighterDto(f.firstName, f.phoneByFighter) " +
>        "FROM EventBidFighter bf JOIN bf.fighter f " +
>        "WHERE bf.eventId = :eventId")
> List<FighterDto> getAllFightersByEventId(@Param("eventId") Integer eventId);
> ```
> Плюсы:
> - Один SELECT.
> - Никакого lazy / persistence context / dirty checking.
> - В выборке только нужные колонки → меньше трафика.
> Минус: DTO нельзя изменить и сохранить обратно (это не entity).

>[!question]- Где N+1 чаще всего стреляет
> - Маппинг ответа REST'а: контроллер вернул entity → Jackson сериализует все поля → дёргает lazy-связи по одной.
> - Цикл по результату списка с обращением к lazy-полям.
> - `toString()` / `equals` / `hashCode`, если в них включены lazy-коллекции.
> - Спецификации Spring Data, которые возвращают entity, а потом сервис проходит по списку и читает связь.

>[!question]- Чек-лист на код-ревью на N+1
> 1. Есть ли цикл по коллекции entity, где внутри читается lazy-связь?
> 2. Контроллер возвращает entity напрямую? (часто скрытая дыра — Jackson дёргает геттеры)
> 3. Если запрос помечен `nativeQuery = true` — связи будут lazy, проверь как используется результат.
> 4. Есть ли `JOIN FETCH` / `@EntityGraph` / `@BatchSize` там, где результат потом обходится с обращением к связям?
> 5. Включи `hibernate.generate_statistics=true` в дев-профиле и смотри Session Metrics.

>[!question]- К equals/hashCode N+1 имеет отношение
> **Нет.** Это отдельная проблема — стратегия загрузки. Решается через JOIN FETCH / EntityGraph / BatchSize. `@EqualsAndHashCode` или его отсутствие на количество запросов не влияет.
> Однако если в `equals`/`hashCode`/`toString` Lombok включит lazy-коллекции — это **триггерит** N+1 на ровном месте, и тогда `@EqualsAndHashCode.Exclude` / `@ToString.Exclude` на коллекциях критичны.

## Смежные вопросы про JOIN

>[!question]- Неявный inner join при выборке — что происходит, если связанная сущность в проекции равна null
> ```java
> @Query("SELECT new com.example.dto.CityDto(c.country, c.region) " +
>        "FROM Event e " +
>        "WHERE e.eventId = :eventId")
> ```
> Если `e.country` — это **объект-сущность** (`@ManyToOne`), а не примитив/int, и он `null` для какой-то строки, эта строка **выпадет из выборки** — Hibernate генерит `INNER JOIN` неявно, как только в проекции/условии участвует поле связанной сущности.
>
> Решение — явный `LEFT JOIN`:
> ```java
> @Query("SELECT new com.example.dto.EventPageDto(" +
>        "e.nameEvent, e.aboutEvent, c.name) " +
>        "FROM Event e " +
>        "LEFT JOIN Country c ON e.country.id = c.id " +
>        "WHERE e.eventId = :eventId")
> Optional<EventPageDto> getEventPageDto(@Param("eventId") Integer eventId);
> ```
> Если же `e.country` — не сущность, а обычный int/примитив (просто колонка), неявного JOIN не возникает вообще: примитивные поля не тянут связанные таблицы.

>[!question]- `LEFT OUTER JOIN` без FETCH — когда полезен
> ```java
> LEFT OUTER JOIN FighterRating fr ON b.idBattle = fr.idBattle
> ```
> Возвращает **все** строки `Battle`, даже если для battle нет ни одной записи в `FighterRating` (в этом случае поля `fr.*` будут `null`). Обычный `JOIN`/`INNER JOIN` такие battle-строки бы выкинул. Используется, когда нужна вся левая таблица независимо от наличия связи, а не для инициализации lazy-ассоциации (для этого — `JOIN FETCH`, см. выше).

>[!question]- `JOIN` без указания типа — это всегда `INNER JOIN`?
> Да, в JPQL (как и в обычном SQL) голый `JOIN` — синоним `INNER JOIN`. Чтобы сохранить строки без совпадения, нужно писать `LEFT JOIN` (`LEFT OUTER JOIN`) явно.

>[!question]- `@Modifying @Query("DELETE FROM ...")` быстрее, чем `findAll` + `deleteAll` — и что при этом теряется
> ```java
> @Modifying
> @Query("DELETE FROM FighterRating f WHERE f.battleId = :battleId")
> void deleteByBattleId(@Param("battleId") Long battleId);
> ```
> vs
> ```java
> List<FighterRating> ratings = fighterRatingRepository.findRatesByBattleId(battleId);
> fighterRatingRepository.deleteAll(ratings);
> ```
> `@Modifying`-запрос — **один SQL `DELETE`**, без загрузки сущностей в persistence context. Быстрее и не даёт N+1 при удалении.
> Цена: он **обходит Hibernate полностью** — `cascade = CascadeType.ALL`, `orphanRemoval = true`, `@PreRemove`/`@PostRemove` колбэки и dirty checking **не срабатывают**, потому что удаление идёт прямым SQL мимо persistence context. Каскадное удаление связанных строк через `@Modifying` тоже не произойдёт — только через явный SQL/FK `ON DELETE CASCADE`, либо через обычный `deleteAll` по managed-сущностям. См. также пример из реальной ошибки в Spring ERROR (Cannot delete or update a parent row).

>[!question]- Резюме одним абзацем
> N+1 = `1 + N` SELECT'ов вместо одного с JOIN'ом. Возникает, когда после загрузки родительской сущности код в цикле трогает lazy-связи. В реальном проекте мгновенно превращает быстрый эндпоинт в десятки секунд. Лечится **на уровне запроса**: `JOIN FETCH` в JPQL, `@EntityGraph` декларативно, `@BatchSize` для компромисса. Native-запросы и `fetch = EAGER` — НЕ решения, а способы переложить проблему в другое место. Включите `hibernate.generate_statistics=true` — Session Metrics покажет реальное число SQL'ей и сразу выявит N+1.