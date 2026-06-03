# Sequences — аналог Stream API в Kotlin

В Kotlin **две модели** обработки коллекций:
- **Eager** — функции на `Iterable` (`map`, `filter` и т.д.) выполняются сразу, создают промежуточные коллекции.
- **Lazy** — `Sequence` — аналог Java `Stream`, ленивые вычисления.

---

>[!question]- BASE — отличие от Java Stream
>1. **`list.map { }.filter { }`** на коллекции — **eager**, создаёт промежуточный список. Это главное отличие от Java Stream!
>2. **`list.asSequence().map { }.filter { }`** — **lazy**, как Stream.
>3. Промежуточных коллекций нет → `Sequence` пропускает элементы pipeline один за другим.
>4. Терминальная операция запускает цепочку: `toList`, `toSet`, `count`, `first`, `forEach`, `reduce`, `sum`...
>5. Получить Sequence: `list.asSequence()`, `sequenceOf(1, 2)`, `generateSequence { ... }`, `sequence { yield(1) }`.

>[!question]- когда использовать Sequence vs обычные функции
>**Sequence выгоднее, когда:**
>- Цепочка длинная (несколько `map`/`filter`/`flatMap` подряд).
>- Источник большой, а нужна только часть (`first`, `take(n)`).
>- Бесконечный источник.
>
>**Обычные функции выгоднее, когда:**
>- Цепочка короткая (1-2 шага) и коллекция небольшая.
>- Нужно несколько раз обращаться к результату.

>[!question]- что не сохраняется в Sequence
>В операциях pipeline ничего не хранится — `map`, `filter`, `take`, `drop`, `flatMap` (stateless).
>**Stateful операции** требуют буфер: `sorted`, `distinct`, `groupBy`.

>[!question]- pipeline модель — поэлементная обработка
>Элемент проходит **всю цепочку** до конца, потом следующий:
>```kotlin
>listOf(1, 2, 3, 4).asSequence()
>    .filter { println("filter $it"); it % 2 == 0 }
>    .map { println("map $it"); it * 10 }
>    .forEach { println("res $it") }
>// filter 1, filter 2, map 2, res 20, filter 3, filter 4, map 4, res 40
>```

>[!question]- short-circuiting
>`first`, `firstOrNull`, `take(n)`, `find`, `any`, `none`, `all` — обрывают цепочку.
>```kotlin
>seq.filter { ... }.map { ... }.first()   // обработает минимум элементов
>```

>[!question]- бесконечные последовательности
>```kotlin
>generateSequence(0) { it + 1 }              // 0, 1, 2, ...
>generateSequence { Random.nextInt() }       // бесконечный random
>generateSequence(startDate) { it.plusMonths(1) }   // даты
>
>// нужен take или first чтобы получить результат
>generateSequence(0) { it + 1 }.take(10).toList()
>```

>[!question]- sequence builder через yield
>```kotlin
>val fib = sequence {
>    var a = 0; var b = 1
>    while (true) {
>        yield(a)
>        val next = a + b; a = b; b = next
>    }
>}
>fib.take(10).toList()   // [0,1,1,2,3,5,8,13,21,34]
>```

---

# Базовые операции

>[!question]- map
>```kotlin
>list.map { it * 2 }
>list.mapIndexed { i, v -> "$i:$v" }
>list.mapNotNull { it.toIntOrNull() }   // отбрасывает null
>```

>[!question]- filter
>```kotlin
>list.filter { it > 0 }
>list.filterNotNull()                    // List<T?> → List<T>
>list.filterIsInstance<String>()         // только String
>list.filterIndexed { i, _ -> i % 2 == 0 }
>```

>[!question]- flatMap
>Разворачивает один уровень вложенности:
>```kotlin
>users.flatMap { it.tags }              // List<List<Tag>> → List<Tag>
>```
>**Несколько уровней** — несколько `flatMap` подряд или рекурсия:
>```kotlin
>fun getAllNames(stage: Stage): Sequence<String> = sequence {
>    yield(stage.name)
>    stage.children.forEach { yieldAll(getAllNames(it)) }
>}
>```

>[!question]- reduce vs fold
>`reduce` — без начального значения, требует непустую коллекцию:
>```kotlin
>list.reduce { acc, x -> acc + x }
>```
>`fold` — с начальным значением, всегда работает, тип результата может отличаться:
>```kotlin
>list.fold(0) { acc, x -> acc + x }
>list.fold(StringBuilder()) { sb, x -> sb.append(x) }
>```

>[!question]- collect-аналоги
>В Kotlin нет `Collectors`, есть terminal-функции:
>```kotlin
>list.toList() / toSet() / toMutableList()
>list.toMap()                              // List<Pair> → Map
>list.associateBy { it.id }                // Map<Id, T>
>list.associate { it.id to it.name }       // Map<Id, Name>
>list.groupBy { it.category }              // Map<K, List<T>>
>list.groupingBy { it.x }.eachCount()      // подсчёт групп
>list.joinToString(", ") { it.name }       // String
>list.sumOf { it.price }
>list.maxByOrNull / minByOrNull
>list.partition { it > 0 }                 // Pair<List, List>
>```

>[!question]- toMap из списка
>```kotlin
>val byId = list.associateBy { it.id }              // First дубль выигрывает
>val byId = list.associate { it.id to it }           // то же
>val byId = list.associateBy({ it.id }, { it.name }) // key, value трансформ
>
>// Если нужны дубликаты — groupBy
>val grouped: Map<Int, List<Item>> = list.groupBy { it.id }
>```

>[!question]- из IntArray в Map
>```kotlin
>nums.withIndex().associate { (i, v) -> i to v }
>// или
>nums.mapIndexed { i, v -> i to v }.toMap()
>```

>[!question]- sorted
>```kotlin
>list.sorted()                            // natural order
>list.sortedBy { it.age }
>list.sortedByDescending { it.age }
>list.sortedWith(compareBy({ it.a }, { it.b }))
>list.sortedWith(compareByDescending<T> { it.a }.thenBy { it.b })
>```

>[!question]- distinct
>```kotlin
>list.distinct()
>list.distinctBy { it.id }
>```

>[!question]- параллельная обработка
>В Kotlin **нет `parallelStream()`**. Для параллелизма используют:
>- **Корутины** + `async` / `awaitAll`:
>```kotlin
>val results = items.map { async { process(it) } }.awaitAll()
>```
>- Java Stream через `list.parallelStream()...` — всё работает.
>- `Flow` для асинхронных потоков данных.

---

>[!question]- side effects и shared state
>Те же риски что в Java parallel streams: shared mutable state ломает корректность.
>**stateless + immutable** — золотое правило для безопасности.
>В корутинах — те же правила.

>[!question]- immutable vs mutable
>**Immutable** — объект не меняется, новый возвращается: `s + "d"`, `list + 1`.
>**Mutable** — состояние меняется: `mutableList.add(x)`, `sb.append(x)`.
>В однопотоке — без разницы. В многопотоке/параллельной обработке mutable shared state = bug.

---

>[!question]- задача без Sequence vs со
>**Без Sequence — eager, промежуточные списки:**
>```kotlin
>val result = list.filter { it > 10 }.sorted().take(3)
>// создаст: отфильтрованный List → отсортированный List → take даст List
>```
>**С Sequence — lazy:**
>```kotlin
>val result = list.asSequence()
>    .filter { it > 10 }
>    .sorted()                  // барьер: материализует
>    .take(3)
>    .toList()
>```

---

>[!question]- Flow vs Sequence
>**Sequence** — синхронный, lazy, одно-поточный.
>**Flow** — асинхронный, lazy, корутины, поддерживает back-pressure. Используется для реактивных потоков.

---

# Шпаргалка

- `list.map.filter` — **eager** в Kotlin (промежуточные списки).
- Для lazy → `list.asSequence().map.filter...toList()`.
- `Sequence` ≈ Java `Stream`.
- Параллелизм — через корутины (`async`/`awaitAll`), не через `parallelStream`.
- `toMap`/`associateBy` — аналог `Collectors.toMap`.
- `groupBy` — аналог `Collectors.groupingBy`.
- `fold` — `reduce` с initial value.
- `flatMap` — один уровень вложенности.
- `sequence { yield(...) }` — кастомный генератор.
