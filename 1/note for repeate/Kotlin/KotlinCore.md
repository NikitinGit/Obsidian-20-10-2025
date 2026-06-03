# BASE Kotlin

>[!question]- Stateless vs Stateful — что это
>Те же CS-понятия что в Java. В Kotlin Sequences: `map`, `filter` — stateless; `sorted`, `distinct` — stateful (барьер).
>В корутинах: stateless suspend-функции легко переносятся между потоками.

>[!question]- Any vs Object
>`Any` — корень иерархии в Kotlin. Содержит только:
>- `equals(other: Any?): Boolean`
>- `hashCode(): Int`
>- `toString(): String`
>
>Нет `wait`, `notify`, `getClass`, `clone`, `finalize`. Для `getClass()` — `obj::class` или `obj::class.java`.
>`Any?` — может быть null, `Any` — нет.

>[!question]- примитивы в Kotlin
>В коде только обёртки: `Byte` (8), `Short` (16), `Int` (32), `Long` (64), `Float` (32), `Double` (64), `Char` (16), `Boolean`.
>На JVM компилятор оптимизирует non-nullable в примитивы. `Int?` остаётся объектом `Integer`.
>**Нет неявных преобразований** примитивов: `val l: Long = 1` — `1` это `Int`, нужно `1L` или `1.toLong()`.

>[!question]- расширения типов 
>```kotlin
>fun Int.toLong()      // явное преобразование
>fun Double.toInt()    // отбрасывает дробную часть, не округляет
>fun Long.toInt()      // берёт младшие 32 бита (как в Java)
>```

>[!question]- хранение чисел с плавающей точкой
>`Float` — IEEE-754 binary32: 1 знак + 8 экспонента + 23 мантисса.
>`Double` — IEEE-754 binary64: 1 знак + 11 экспонента + 52 мантисса.
>`0.1 + 0.2 != 0.3` — для точности `BigDecimal("0.1").add(BigDecimal("0.2"))`.

>[!question]- объявление массивов
>```kotlin
>val arr = intArrayOf(1, 2, 3)       // IntArray (примитив)
>val arr2 = arrayOf(1, 2, 3)         // Array<Int> (обёртка)
>val arr3 = IntArray(5)              // [0,0,0,0,0]
>val arr4 = IntArray(5) { it * 2 }   // [0,2,4,6,8]
>val size = arr.size                  // не .length
>```

>[!question]- классы — открытость по умолчанию
>В Kotlin классы и методы **final по умолчанию**. Чтобы можно было наследовать — `open`:
>```kotlin
>open class Animal {
>    open fun sound() = "..."
>}
>class Dog : Animal() {
>    override fun sound() = "woof"  // override обязателен
>}
>```

>[!question]- видимость по умолчанию
>**public по умолчанию** (а не package в Java).
>Модификаторы: `public`, `internal` (внутри модуля), `protected`, `private`.

>[!question]- конструкторы
>**Первичный конструктор** в заголовке класса:
>```kotlin
>class User(val name: String, var age: Int)  // автополя
>class Foo(name: String) {                    // параметр без val/var — только в init
>    val n = name
>    init { println("created $name") }
>}
>```
>**Вторичные конструкторы** через `constructor`:
>```kotlin
>class User(val name: String) {
>    constructor(name: String, age: Int) : this(name) { ... }
>}
>```

>[!question]- наследование и super
>```kotlin
>open class Parent(val x: Int)
>class Child(x: Int, val y: Int) : Parent(x)   // вызов super-конструктора в заголовке
>
>override fun foo() {
>    super.foo()    // вызов метода родителя
>}
>```

>[!question]- data class
>```kotlin
>data class User(val name: String, val age: Int)
>```
>Автоматически: `equals`, `hashCode`, `toString`, `copy`, `componentN()`.
>```kotlin
>val u2 = u.copy(age = 30)
>val (name, age) = u    // деструктуризация
>```
>Заменяет рукописный POJO/record.

>[!question]- sealed class / interface
>Ограниченная иерархия — все наследники известны компилятору. Аналог Java `sealed`.
>```kotlin
>sealed interface Result<T>
>class Success<T>(val value: T) : Result<T>
>class Error<T>(val msg: String) : Result<T>
>
>when (result) {              // не нужен else — компилятор знает все варианты
>    is Success -> ...
>    is Error -> ...
>}
>```

>[!question]- object / companion object
>**`object`** — синглтон:
>```kotlin
>object Logger { fun log(msg: String) = println(msg) }
>Logger.log("hi")
>```
>**`companion object`** — заменяет `static`:
>```kotlin
>class User {
>    companion object {
>        const val MAX_AGE = 150
>        fun create(name: String) = User(name)
>    }
>}
>User.create("Bob")
>```

>[!question]- == vs ===
>`==` — это `equals()` (с null-проверкой).
>`===` — сравнение ссылок (как `==` в Java).

>[!question]- is / !is / as / as?
>```kotlin
>if (x is String) x.length     // smart cast
>if (x !is Int) return
>val s = x as String             // ClassCastException если не подходит
>val s = x as? String            // null если не подходит — safe cast
>```

>[!question]- when — замена switch
>Выражение, возвращает значение:
>```kotlin
>val res = when (x) {
>    1, 2 -> "small"
>    in 3..10 -> "medium"
>    is String -> "string of length ${x.length}"
>    else -> "unknown"
>}
>when {                        // без аргумента — цепочка условий
>    x > 0 -> "+"
>    x < 0 -> "-"
>    else -> "0"
>}
>```

>[!question]- циклы
>```kotlin
>for (i in 0..10)         // включительно
>for (i in 0 until 10)    // исключая 10
>for (i in 10 downTo 0 step 2)
>for ((index, value) in list.withIndex())
>repeat(5) { println(it) }
>```

>[!question]- функции
>```kotlin
>fun add(a: Int, b: Int): Int = a + b              // single-expression
>fun greet(name: String = "world") = "Hi, $name"   // дефолтные параметры
>greet(name = "Bob")                                // named arguments
>fun sum(vararg nums: Int) = nums.sum()             // varargs
>```

>[!question]- extension function
>Добавляет метод к существующему типу без наследования:
>```kotlin
>fun String.lastChar(): Char = this[length - 1]
>"hello".lastChar()  // 'o'
>```
>Под капотом — статический метод с `this` первым параметром.

>[!question]- string templates
>```kotlin
>val s = "Hello, $name, age=${user.age}"
>val multi = """
>    raw text
>    with $variables
>""".trimIndent()
>```

>[!question]- toString вызывается когда
>`println(obj)`, `"$obj"`, `obj.toString()` явно.

>[!question]- nothing-тип
>`Nothing` — у функций которые никогда не возвращают (`throw`, бесконечный цикл).
>`val x: String = nullable ?: throw E()` — компилятор знает, что после `throw` (тип `Nothing`) `x` non-null.

>[!question]- Unit
>Аналог `void`. Возвращается из функций без `return value`.
>В отличие от void — это реальный тип, можно присвоить переменной.

# Память и JVM
Те же концепции что в Java ([[Kotlin JVM]]):
- Куча (объекты, массивы), стек (локалки, фреймы), метаспейс (классы).
- Stack overflow при глубокой рекурсии. Kotlin поддерживает `tailrec` — оптимизация хвостовой рекурсии:
>```kotlin
>tailrec fun factorial(n: Int, acc: Long = 1): Long =
>    if (n <= 1) acc else factorial(n - 1, acc * n)
>```
>Компилятор превращает в цикл — стек не растёт.

>[!question]- глубина стектрейса
>`Thread.currentThread().stackTrace.size`

# Полиморфизм
Та же истина что и в Java:
- Объект всегда одного типа — того что был создан через конструктор.
- `this` — реальный объект (самый нижний класс).
- `is` проверяет реальный runtime-тип.
- Тип переменной — только "ярлык" для компилятора.

>[!question]- super в Kotlin
>`super.method()` — метод ближайшего родителя.
>При множественной реализации интерфейсов с одинаковым методом:
>```kotlin
>super<InterfaceA>.method()
>```

>[!question]- equals + hashCode контракт
>Тот же что в Java: `a == b` ⇒ `a.hashCode() == b.hashCode()`.
>В `data class` — генерируется автоматически по всем полям первичного конструктора.
>Для JPA-entity — НЕ использовать `data class` (см. [[JAVA-HASHMAP]] раздел про Hibernate proxy).
