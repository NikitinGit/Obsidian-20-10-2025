# Проблема N+1 в JPA / Hibernate (Kotlin)

Полная теория совпадает с Java — см. [[JPA N+1 problem]]. Здесь — Kotlin-специфика.

---

>[!question]- Что такое N+1 — одной фразой
>Hibernate выполняет **1 запрос за родителями + N запросов на каждый дочерний объект** при доступе к lazy-связям в цикле. Решается через `JOIN FETCH`, `@EntityGraph`, `@BatchSize`.

>[!question]- 1+1+N пример в Kotlin
>```kotlin
>val event = eventRepository.findById(eventId).orElseThrow()           // 1 SELECT
>val bids = event.eventBidFighters                                     // 1 SELECT bids
>for (bid in bids) {
>    bid.fighter.firstName                                             // 1 SELECT на каждого
>}
>```
>Итого `1 + 1 + N`.

>[!question]- JPQL JOIN FETCH в Spring Data Kotlin
>```kotlin
>@Query("""
>    SELECT DISTINCT e FROM Event e
>    LEFT JOIN FETCH e.eventBidFighters bf
>    LEFT JOIN FETCH bf.fighter
>    WHERE e.eventId = :eventId
>""")
>fun findEventWithBidsAndFighters(@Param("eventId") eventId: Int): Optional<Event>
>```
>Multi-line string через `"""..."""` идиоматичен.

>[!question]- @EntityGraph
>```kotlin
>@EntityGraph(attributePaths = ["eventBidFighters", "eventBidFighters.fighter"])
>@Query("SELECT e FROM Event e WHERE e.eventId = :eventId")
>fun findEventByEventIdGraph(@Param("eventId") eventId: Int): Optional<Event>
>```

>[!question]- @BatchSize
>```kotlin
>@BatchSize(size = 50)
>@ManyToOne(fetch = FetchType.LAZY)
>lateinit var fighter: Fighter
>```

>[!question]- nativeQuery не спасает
>```kotlin
>@Query(value = "SELECT * FROM events WHERE event_id = :eventId", nativeQuery = true)
>fun findRaw(@Param("eventId") eventId: Int): Optional<Event>
>```
>Native тащит только колонки таблицы, связи остаются lazy → N+1 как было.

>[!question]- @ManyToOne(fetch = EAGER) — почему плохо
>EAGER тащит связь в **каждом** запросе родителя. Стратегия должна быть LAZY, а fetch — на конкретный запрос через JOIN FETCH/EntityGraph.

>[!question]- MultipleBagFetchException
>Двойной `JOIN FETCH` к коллекциям типа `List` (`Bag`):
>```
>cannot simultaneously fetch multiple bags
>```
>Решения:
>- Заменить одну `List` на `Set` (`MutableSet`).
>- Два отдельных запроса.

>[!question]- DTO-проекция через конструктор
>```kotlin
>data class FighterDto(val firstName: String, val phone: String?)
>
>@Query("""
>    SELECT new com.example.FighterDto(f.firstName, f.phoneByFighter)
>    FROM EventBidFighter bf JOIN bf.fighter f
>    WHERE bf.eventId = :eventId
>""")
>fun getAllFightersByEventId(@Param("eventId") eventId: Int): List<FighterDto>
>```
>**Важно:** для конструктора в JPQL нужен **non-data конструктор** или JPQL-совместимая сигнатура. `data class` обычно работает.

>[!question]- JPA-entity в Kotlin — best practices
>- **Не делать `data class`** — `equals`/`hashCode` по всем полям дёрнут lazy-коллекции.
>- Поля как `var`, не `val` (Hibernate использует setter'ы / рефлексию).
>- Открыть класс — `open` или плагин `kotlin-jpa` (делает классы и поля open автоматически).
>- Для `lateinit var` — нельзя у примитивов и nullable. Для них использовать `var x: Int = 0`.
>- Нет конструктора по умолчанию → плагин `kotlin-jpa` добавляет no-arg конструктор автоматически.
>
>```kotlin
>// build.gradle.kts
>plugins {
>    kotlin("plugin.jpa") version "..."
>    kotlin("plugin.spring") version "..."
>}
>```
>
>```kotlin
>@Entity
>class Event(
>    @Id @GeneratedValue var eventId: Int? = null,
>    var name: String = "",
>    @OneToMany(mappedBy = "event", fetch = FetchType.LAZY)
>    var bids: MutableList<EventBidFighter> = mutableListOf()
>) {
>    override fun equals(other: Any?): Boolean {
>        if (this === other) return true
>        val oCls = if (other is HibernateProxy)
>            other.hibernateLazyInitializer.persistentClass
>        else other?.javaClass ?: return false
>        val thisCls = if (this is HibernateProxy)
>            (this as HibernateProxy).hibernateLazyInitializer.persistentClass
>        else javaClass
>        if (thisCls != oCls) return false
>        other as Event
>        return eventId != null && eventId == other.eventId
>    }
>    override fun hashCode(): Int = javaClass.hashCode()
>}
>```

>[!question]- HHH000104: firstResult/maxResults с collection fetch
>Hibernate грузит ВСЮ выборку в память и режет постранично там. Решения:
>1. Два запроса: пагинация по id, потом `JOIN FETCH WHERE id IN`.
>2. `@BatchSize` вместо JOIN FETCH.

>[!question]- Где N+1 чаще всего стреляет
>- Контроллер возвращает entity → Jackson дёргает геттеры → lazy инициализация.
>- Цикл по результату с обращением к lazy-полю.
>- `equals/hashCode/toString` Lombok/data class включают lazy-коллекции.
>- Spring Data возвращает entity, сервис обходит список с обращением к связям.

>[!question]- Чек-лист на код-ревью
>1. Цикл по entity с чтением lazy-связи?
>2. Контроллер возвращает entity напрямую?
>3. `nativeQuery = true` — связи lazy.
>4. Есть `JOIN FETCH`/`@EntityGraph`/`@BatchSize` где нужно?
>5. `hibernate.generate_statistics=true` → Session Metrics в логах.

>[!question]- Резюме
>N+1 = `1 + N` SELECT'ов вместо одного с JOIN'ом. Лечится на уровне запроса: `JOIN FETCH`, `@EntityGraph`, `@BatchSize`. `nativeQuery` и `EAGER` — не решения. В Kotlin особенно: **не использовать `data class` для entity**, использовать плагин `kotlin-jpa`.
