# Лямбда-выражения в Kotlin

>[!question]- синтаксис лямбды
>```kotlin
>val sum = { a: Int, b: Int -> a + b }
>sum(1, 2)                          // 3
>val isEven: (Int) -> Boolean = { it % 2 == 0 }
>```
>В фигурных скобках, параметры до `->`, тело после. Тип выводится — часто типы параметров можно опустить.

>[!question]- it — implicit parameter
>Если параметр один — называется `it` без объявления:
>```kotlin
>list.filter { it > 0 }
>list.map { it * 2 }
>```
>Если параметров несколько — нужно объявить: `{ a, b -> a + b }`.

>[!question]- trailing lambda
>Если последний параметр функции — лямбда, её можно вынести за скобки:
>```kotlin
>list.map({ it * 2 })       // обычный синтаксис
>list.map { it * 2 }        // trailing lambda — идиоматично
>list.fold(0) { acc, x -> acc + x }
>```
>Если лямбда — единственный аргумент, скобки не нужны вообще.

>[!question]- функциональные типы
>```kotlin
>(Int) -> Int               // функция Int → Int
>(Int, Int) -> Int          // (Int, Int) → Int
>() -> Unit                 // без параметров, void
>(String) -> Unit
>suspend (Int) -> Int       // suspend-функция (корутины)
>```
>Под капотом — интерфейсы `Function0`, `Function1`, ... `Function22`.

>[!question]- замыкания (closures)
>Лямбда захватывает переменные из внешней области видимости.
>**В отличие от Java** — можно захватывать **`var` тоже**, не только effectively final:
>```kotlin
>var counter = 0
>list.forEach { counter++ }      // OK, в Java было бы ошибкой
>```
>Под капотом — оборачивается в `Ref.IntRef`/`ObjectRef`.

>[!question]- method references
>```kotlin
>list.map(String::uppercase)        // = { it.uppercase() }
>list.map(::parseInt)                // top-level/static method
>list.map(this::process)             // instance method
>list.sortedWith(compareBy(User::age))
>```

>[!question]- инлайн-функции
>Лямбда обычно создаёт объект (Function1 и т.д.) → нагрузка на GC.
>`inline` подставляет тело функции и лямбду в место вызова:
>```kotlin
>inline fun <T> measureTime(block: () -> T): T {
>    val start = System.nanoTime()
>    val r = block()
>    println("took ${System.nanoTime() - start}")
>    return r
>}
>```
>**Бонус:** в inline функциях работает:
>- `return` из лямбды выходит из внешней функции (non-local return).
>- Можно использовать `reified T`.

>[!question]- non-local return
>```kotlin
>fun findFirst(list: List<Int>): Int? {
>    list.forEach {                  // forEach inline
>        if (it > 10) return it      // выходит из findFirst, не из лямбды
>    }
>    return null
>}
>```
>Без inline — return только из лямбды нельзя (нужен `return@forEach`).

>[!question]- labeled return
>```kotlin
>list.forEach {
>    if (it < 0) return@forEach      // продолжаем итерацию
>    println(it)
>}
>list.forEach label@{
>    if (it < 0) return@label
>}
>```

>[!question]- crossinline / noinline
>`noinline` — параметр лямбды, который нельзя инлайнить (например, передаём дальше).
>`crossinline` — лямбда инлайнится, но non-local return запрещён (используется внутри обёртки/потока).

>[!question]- SAM-преобразования
>Лямбду можно передавать в Java-интерфейсы с одним методом (SAM):
>```kotlin
>val r = Runnable { println("hi") }
>thread.start(Runnable { ... })       // или просто { ... } после Java 8 синтаксиса
>```
>Для Kotlin-интерфейсов нужно явно объявить `fun interface`:
>```kotlin
>fun interface Predicate<T> { fun test(t: T): Boolean }
>val p: Predicate<Int> = Predicate { it > 0 }
>val p: Predicate<Int> = { it > 0 }   // тоже работает
>```

>[!question]- лямбда с receiver
>Тип `T.() -> R` — лямбда с приёмником. Внутри `this` — экземпляр `T`:
>```kotlin
>val build: StringBuilder.() -> Unit = { append("hi"); append("!") }
>StringBuilder().build()
>
>// DSL-стиль:
>fun html(block: HtmlBuilder.() -> Unit): String { ... }
>html {
>    body { p("hello") }
>}
>```

---

# Scope-функции — let, run, with, apply, also

| Функция | Receiver | Returns | Применение |
|---|---|---|---|
| `let` | `it` | результат лямбды | трансформация, null-check |
| `run` | `this` | результат лямбды | конфигурация + возврат значения |
| `with` | `this` | результат лямбды | вызов нескольких методов на объекте |
| `apply` | `this` | сам объект | конфигурация и возврат того же объекта |
| `also` | `it` | сам объект | side effect, логи |

>[!question]- let — null-safe
>```kotlin
>user?.let { println(it.name) }       // выполнится только если user != null
>val len = name?.let { it.length } ?: 0
>```

>[!question]- apply — конфигурация
>```kotlin
>val user = User().apply {
>    name = "Alice"
>    age = 30
>}                                       // возвращает User
>```

>[!question]- also — side effect
>```kotlin
>val user = createUser().also { log.info("created $it") }
>```

>[!question]- run — комбо
>```kotlin
>val len = "hello".run { uppercase().length }
>val r = run { ... ; result }            // без receiver — просто блок
>```

>[!question]- with — вызовы на объекте
>```kotlin
>val res = with(builder) {
>    append("hello")
>    append("!")
>    toString()
>}
>```

---

# Функциональные интерфейсы из stdlib

```kotlin
(T) -> R                    // = Function<T, R>
(T) -> Unit                 // = Consumer<T>
() -> T                     // = Supplier<T>
(T) -> Boolean              // = Predicate<T>
(T, U) -> R                 // = BiFunction<T, U, R>
```

В Kotlin их редко используют напрямую — обычно функциональные типы пишут как `(T) -> R`.

---

# Шпаргалка

- Лямбды в `{ }`, `->` отделяет параметры.
- `it` — implicit param если один.
- Trailing lambda — `f { ... }` чище чем `f({ ... })`.
- Можно захватывать `var` (в отличие от Java).
- `inline` — для производительности и `reified` / non-local return.
- Метод-references через `::`.
- Scope-функции (`let`, `apply`, `also`, `run`, `with`) — выбор по нужному `this`/`it` и возвращаемому значению.
- Лямбды с receiver — основа DSL.
