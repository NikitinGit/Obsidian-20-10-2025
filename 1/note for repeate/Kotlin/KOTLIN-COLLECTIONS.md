# Памятка по коллекциям, HashMap и контракту equals/hashCode

---

## 1. Mutable vs Immutable интерфейсы

Kotlin разделяет коллекции на **read-only** и **mutable** на уровне интерфейсов:

| Read-only | Mutable |
|---|---|
| `List<T>` | `MutableList<T>` |
| `Set<T>` | `MutableSet<T>` |
| `Map<K, V>` | `MutableMap<K, V>` |

```kotlin
val list = listOf(1, 2, 3)              // List<Int>, неизменяемый
val mutable = mutableListOf(1, 2, 3)    // MutableList<Int>
mutable.add(4)
// list.add(4)                           // нет метода add у List
```

**Важно:** read-only ≠ immutable. Под капотом часто `ArrayList`, который можно изменить через Java-API. Это **view**, не гарантированная immutability.

---

## 2. Создание коллекций

```kotlin
listOf(1, 2, 3)              // List, immutable view
mutableListOf(1, 2, 3)       // MutableList
arrayListOf(1, 2, 3)         // ArrayList
setOf("a", "b")              // Set
mutableSetOf("a")
mapOf("k" to 1, "k2" to 2)   // Map
mutableMapOf("k" to 1)
emptyList<Int>()
buildList { add(1); add(2) } // DSL builder
```

`to` — extension инфиксная функция, создаёт `Pair`.

---

## 3. HashMap — устройство

**То же что в Java**: массив бакетов, `spread(hash) & (cap - 1)`, treeify при ≥ 8 в бакете и capacity ≥ 64.

```kotlin
val m = HashMap<String, Int>()        // java.util.HashMap под капотом
val m = hashMapOf("a" to 1)
val m = mutableMapOf("a" to 1)        // обычно LinkedHashMap
```

| Параметр | По умолчанию |
|---|---|
| capacity | 16 |
| loadFactor | 0.75 |
| threshold | 12 |
| TREEIFY_THRESHOLD | 8 |
| MIN_TREEIFY_CAPACITY | 64 |

Все детали — см. [[JAVA-HASHMAP]] (применимо целиком).

---

## 4. Контракт equals / hashCode

Те же правила что в Java:
1. `a == b` ⇒ `a.hashCode() == b.hashCode()`.
2. Поля, по которым считается hash, не должны меняться пока объект в коллекции.

### data class — генерирует автоматически
```kotlin
data class User(val id: Int, val name: String)
// equals/hashCode по всем полям первичного конструктора
```

### Когда data class опасен
**JPA-entity** — не делать `data class`:
- `equals`/`hashCode` по всем полям → дёргает lazy-коллекции → N+1.
- `hashCode` меняется при `persist` (id : null → 42) → объект "теряется" в Set.

Решение — рукописный equals по id с `is`-проверкой через `HibernateProxy` ([[JAVA-HASHMAP]] раздел 9-10):
```kotlin
override fun equals(other: Any?): Boolean {
    if (this === other) return true
    if (other == null) return false
    val oCls = if (other is HibernateProxy)
        other.hibernateLazyInitializer.persistentClass
    else other::class.java
    val thisCls = if (this is HibernateProxy)
        (this as HibernateProxy).hibernateLazyInitializer.persistentClass
    else this::class.java
    if (thisCls != oCls) return false
    other as Event
    return eventId != null && eventId == other.eventId
}

override fun hashCode(): Int = this::class.hashCode()
```

---

## 5. Set из коллекций

`HashSet`, `LinkedHashSet`, `TreeSet` — всё работает как в Java.
- `setOf(...)` → `LinkedHashSet` (порядок вставки)
- `mutableSetOf(...)` → `LinkedHashSet`
- `hashSetOf(...)` → `HashSet`

```kotlin
val s = sortedSetOf(3, 1, 2)   // TreeSet
```

---

## 6. Полезные функции коллекций

```kotlin
list.first { it > 0 }
list.firstOrNull { it > 0 }    // null если нет
list.find { it > 0 }            // = firstOrNull
list.any { it > 0 }
list.all { it > 0 }
list.none { it > 0 }
list.count { it > 0 }
list.sumOf { it.price }
list.maxByOrNull { it.age }
list.minByOrNull { it.age }
list.groupBy { it.category }    // Map<K, List<V>>
list.associateBy { it.id }      // Map<K, V>
list.associate { it.id to it.name }
list.partition { it > 0 }       // Pair<List, List>
list.zip(other)                  // List<Pair>
list.windowed(3, 1)
list.chunked(3)
list.distinct()
list.distinctBy { it.id }
list.sortedBy { it.name }
list.sortedByDescending { it.age }
list.flatMap { it.tags }
list.fold(0) { acc, x -> acc + x }   // = reduce с initial
list.reduce { acc, x -> acc + x }
```

---

## 7. Сложность операций HashMap

Та же что в Java (см. [[JAVA-HASHMAP]] раздел 8):
- `get`/`put`/`remove` — amortized `O(1)`.
- В худшем случае с treeify — `O(log n)`.
- Без хорошего `hashCode` — деградация до `O(n)`.

---

## 8. Шпаргалка

- Read-only коллекции (`List`, `Set`, `Map`) ≠ immutable — это интерфейс без методов модификации.
- `data class` — `equals`/`hashCode` бесплатно, но **не для JPA-entity**.
- HashMap внутри — `java.util.HashMap`, все правила Java применимы.
- Для immutable view — `listOf()`, `mapOf()`, или `Collections.unmodifiableXxx()`.
- Для JPA-entity использовать рукописный `equals` по бизнес-ключу или id, `hashCode` = `class.hashCode()` стабильный.
