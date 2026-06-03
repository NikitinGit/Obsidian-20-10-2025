>[!question]- Kotlin компиляция
> Исходник `.kt` → байткод `.class` (тот же JVM bytecode что и у Java) → JVM выполняет.
> Компилятор `kotlinc`. Также есть Kotlin/Native (LLVM) и Kotlin/JS.

>[!question]- JVM
> Виртуальная машина, интерпретирует байт-код, через JIT компилирует горячие места в машинный код.
> 2 JIT: C1 (быстрый запуск, флаг `-client`) и C2 (`-server`, более агрессивная оптимизация). На x64 `-server` по умолчанию.
> HotSpot — самая известная реализация JVM.

>[!question]- classpath
> Список путей где JVM ищет `.class`-файлы и ресурсы.
> ```
> java -cp /path/to/classes:/path/to/lib.jar MainKt
> ```
> Kotlin-файл `Main.kt` с top-level `fun main()` компилируется в класс `MainKt`.

>[!question]- bytecode
> Промежуточный код, не зависит от ОС. Инструкции для JVM.
> Kotlin-фишки (data class, lambda, smart cast, extension) превращаются в обычные классы и методы.

>[!question]- JDK / JRE
>**JDK** — набор разработчика: JRE + компилятор + инструменты.
>**JRE** — среда исполнения: JVM + библиотеки. Без компилятора.
>Для Kotlin нужен **JDK** (kotlinc запускается на JVM) + Kotlin compiler.

>[!question]- Память в JVM
>Та же что для Java:
>- **Стек** — локальные переменные, параметры метода, адрес возврата, фреймы.
>- **Куча** — объекты, массивы, строки (всё что через `new`/конструктор).
>- **Метаспейс** — классы (не входит в Xmx).

>[!question]- что хранит куча
>Объекты, массивы, коллекции, строки. Всё что создаётся через конструктор.
>```kotlin
>fun main() {
>    val a = 10                  // примитив в стеке (на JVM int)
>    val s = "hello"             // String pool
>    val arr = IntArray(5)       // объект в куче
>    val user = User("Alice")    // объект в куче
>}
>```

>[!question]- StackOverflowError
>Глубокая рекурсия. Решается через `tailrec` (компилятор схлопывает в цикл):
>```kotlin
>tailrec fun factorial(n: Int, acc: Long = 1): Long =
>    if (n <= 1) acc else factorial(n - 1, acc * n)
>```

>[!question]- увеличить стек
>```
>java -Xss512k MainKt
>```
>IntelliJ → Run > Edit Configurations > VM options → `-Xss2m`

>[!question]- размер кучи
>```
>java -Xms256m -Xmx2g MainKt
>```

>[!question]- глубина стектрейса
>```kotlin
>Thread.currentThread().stackTrace.size
>```

>[!question]- top-level функции — как они выглядят в bytecode
>Top-level `fun main()` в файле `Main.kt` становится `public static void main()` в классе `MainKt`.
>Можно настроить через `@file:JvmName("MyMain")`.

>[!question]- companion object — как в bytecode
>Поля и методы становятся static с доступом через статическое поле `Companion`:
>```kotlin
>class Foo {
>    companion object { const val X = 1 }
>}
>// Java: Foo.Companion.getX() или Foo.X для @JvmField/const
>```
>`@JvmStatic` делает метод реально статическим.

>[!question]- сборщики мусора (GC)
>Те же что в Java: Serial, Parallel, G1 (по умолчанию с JDK 9), ZGC, Shenandoah.
>Для Kotlin важно: data class, лямбды, корутины создают много мелких объектов → G1/ZGC лучше Parallel.
