# Памятка по типам и кастам в Kotlin

---

## 1. ClassCastException — как в Java

```kotlin
val an: Animal = Animal()
val dog = an as Dog          // ClassCastException
val dog = an as? Dog         // null, не падает
```

**Safe-cast паттерн** через smart cast:
```kotlin
if (an is Dog) {
    an.bark()                // an автоматически Dog здесь
}
```

`as?` возвращает `null` вместо исключения — идиоматический Kotlin-вариант.

---

## 2. Класс vs Тип

Та же истина что в Java: **Класс ≠ Тип**.

### Что есть тип, но не класс
- **Все типы в Kotlin — объектные** (нет примитивов в коде), но `Int`, `Long` компилируются в JVM-примитивы где можно.
- **Nullable**: `String?` — отдельный тип от `String`.
- **Интерфейсы**: `Runnable`, `Comparable<T>`.
- **Массивы**: `IntArray`, `Array<Dog>`.
- **Параметризованные**: `List<String>` ≠ `List<Int>`.
- **Star-projection**: `List<*>` — аналог `List<?>`.
- **Type variables**: `T`, `E`.
- **Nothing**, **Unit** — собственные типы.

### Несколько типов у одного объекта
```kotlin
val d = Dog()
val a: Animal = d       // тот же объект, тип Animal
val o: Any = d          // тип Any
val r: Runnable = d     // если Dog : Runnable
```

---

## 3. Виды кастов в Kotlin

### 3.1 По направлению
| Каст | Описание | Пример |
|---|---|---|
| **Upcast** | к супертипу, неявный | `val a: Animal = Dog()` |
| **Downcast** | к подтипу, явный | `val d = a as Dog` |
| **Safe cast** | null если не подходит | `val d = a as? Dog` |
| **Smart cast** | автоматически после `is` | `if (a is Dog) a.bark()` |

### 3.2 Преобразование примитивов
**Нет неявных преобразований!** В Java `int → long` неявно, в Kotlin — нет:
```kotlin
val i: Int = 10
val l: Long = i              // ОШИБКА
val l: Long = i.toLong()     // OK
```
Методы: `toInt()`, `toLong()`, `toDouble()`, `toFloat()`, `toShort()`, `toByte()`, `toChar()`.

### 3.3 Boxing
Происходит автоматически когда нужен nullable или generic:
```kotlin
val a: Int = 5       // примитив int
val b: Int? = 5      // boxed Integer
val list: List<Int> = listOf(1, 2)   // List<Integer> на JVM
```

---

## 4. Nullable-типы

`T?` = `T` или `null`. Полностью разные типы для компилятора.

```kotlin
var s: String = "hi"
s = null              // ОШИБКА компиляции

var s: String? = "hi"
s = null              // OK
s.length              // ОШИБКА — нужен safe call или !!
s?.length             // OK, возвращает Int?
s!!.length            // KotlinNullPointerException если null
s?.length ?: 0        // элвис
```

### Smart cast после null-check
```kotlin
if (s != null) {
    s.length    // s : String внутри блока
}
```

### Platform types
Типы из Java: `String!` — может быть и `null`, и не быть. Компилятор не проверяет. **Опасное место** на стыке Java/Kotlin.

---

## 5. Массивы как типы

Аналогично Java: массив — полноценный тип, у каждой комбинации `Class`-объект.

```kotlin
IntArray(5)              // на JVM = int[]
Array<Int>(5) { 0 }      // на JVM = Integer[]
Array<Dog>(0) { ... }    // = Dog[]
```

**Массивы ковариантны** (как в Java): `Array<Dog>` is-a `Array<Animal>` → `ArrayStoreException` в runtime.
Но для **`IntArray`** etc — они не наследуют общий тип, инвариантны.

---

## 6. Generics-типы

### Erasure — как в Java
```kotlin
val a = ArrayList<String>()
val b = ArrayList<Int>()
a::class == b::class    // true — один Class
```

### Reified type parameter (только inline-функции)
В отличие от Java, Kotlin позволяет получить тип в runtime через `reified`:
```kotlin
inline fun <reified T> isInstance(x: Any) = x is T   // работает!
inline fun <reified T> Gson.fromJson(json: String): T = fromJson(json, T::class.java)
```
Компилятор подставляет конкретный тип в место вызова — type erasure обходится.

### Кирпичики
| Конструкция | Что значит |
|---|---|
| `List<String>` | параметризованный |
| `List<*>` | star-projection (аналог `List<?>`) |
| `List<out Animal>` | ковариантность (≈ `? extends Animal`) |
| `List<in Dog>` | контравариантность (≈ `? super Dog`) |
| `<T : Animal>` | type variable с границей |
| `reified T` | сохраняется в runtime в inline-функциях |

---

## 7. Вариативность — `in` / `out`

Kotlin делает variance **на уровне объявления типа** (declaration-site variance), а не на каждом использовании как в Java (use-site).

### `out T` — ковариантность (producer)
Тип `T` может **только возвращаться** из методов:
```kotlin
interface Producer<out T> {
    fun get(): T          // OK
    // fun set(t: T)       // ОШИБКА — T в позиции in
}
val p: Producer<Animal> = producer  // Producer<Dog> совместим
```

### `in T` — контравариантность (consumer)
Тип `T` может **только приниматься** в параметрах:
```kotlin
interface Consumer<in T> {
    fun accept(t: T)
    // fun get(): T        // ОШИБКА
}
val c: Consumer<Dog> = animalConsumer  // Consumer<Animal> совместим
```

### Use-site variance (как Java wildcards)
```kotlin
fun copy(from: Array<out Any>, to: Array<Any>) { ... }   // ≈ Array<? extends Any>
```

### PECS в терминах Kotlin
- **out** = `? extends T` — producer extends
- **in** = `? super T` — consumer super

### Стандартная библиотека
- `List<out T>` — иммутабельный, ковариантный
- `MutableList<T>` — инвариантный (может и принимать, и отдавать)
- `Function1<in P, out R>` — контравариантен по входу, ковариантен по выходу

```kotlin
val animals: List<Animal> = listOf<Dog>(Dog())   // OK — List ковариантен по out T
val mutable: MutableList<Animal> = mutableListOf<Dog>()  // ОШИБКА — инвариантен
```

### Массивы vs Generics
| | Массивы Kotlin | Generics Kotlin |
|---|---|---|
| Вариативность | `Array<T>` ковариантен (Java наследие) | `List<T>` инвариантен, `out`/`in` для variance |
| Проверка | runtime `ArrayStoreException` | compile-time |
| Erasure | у каждого свой `Class` | общий `Class`, кроме `reified` |
| Управление | автоматически | `out`/`in` или `out`/`in` на сайте |

---

## 8. when + pattern matching

Аналог Java switch pattern matching, работает из коробки:
```kotlin
when (an) {
    is Dog -> an.bark()
    is Cat -> an.meow()
    is Snake -> an.hiss()
    else -> { }
}
```

С `sealed` — exhaustive, `else` не нужен:
```kotlin
sealed interface Animal
class Dog : Animal
class Cat : Animal

when (an) {              // компилятор знает все варианты
    is Dog -> ...
    is Cat -> ...
}
```

---

## 9. Шпаргалка-резюме

- **Каст падает** только когда фактический тип не совместим. `as?` — null вместо исключения.
- **Safe downcast** — smart cast после `is`, или `as?`.
- **Nullable** — отдельный тип: `String` ≠ `String?`.
- **Нет неявного преобразования примитивов** — `5.toLong()`, не `5L`.
- **Generics инвариантны** по умолчанию, **`out`/`in`** для variance на сайте объявления.
- **`reified`** в inline-функциях обходит type erasure.
- **`when` + `is`** — pattern matching из коробки, exhaustive с `sealed`.
