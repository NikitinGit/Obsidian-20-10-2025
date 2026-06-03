>[!question]- какая типизация у котлина и чем отличается от джава
> В **Java** любое ссылочное значение может быть `null`, и это часто ведёт к `NullPointerException` (_NPE_).
> В **Kotlin** типовая система разделяет:
> - **nullable типы**: `String?` (может быть `null`)  
> - **non-null типы**: `String` (не может быть `null`)

>[!question]- объявить переменную 
>var 

>[!question]- объявить константу 
>val

>[!question]- безопасный вызов 
>`a?.b` — если `a == null`, возвращает `null`, иначе `a.b`
>`a?.b?.c?.d` — цепочка безопасных вызовов 

>[!question]- элвис оператор 
>`a ?: default` — если `a == null`, вернёт `default`
>часто используют с `throw`: `val x = nullable ?: throw IllegalStateException()`

>[!question]- !! оператор 
>`a!!` — явное утверждение что `a != null`. Если null — `KotlinNullPointerException`.
>Использовать редко, только когда уверен на 100%.

>[!question]- smart cast 
>после проверки `if (x is String)` или `x != null` — внутри блока `x` уже как `String` без явного каста. Работает для `val` и локальных `var`.

>[!question]- основные отличия от Java
>- нет `new` — `User("Alice")`
>- `;` не нужен
>- всё выражение: `val x = if (...) 1 else 2`
>- по умолчанию классы и методы `final` (нужен `open` для наследования)
>- нет `static` — заменяют `companion object` / top-level функции
>- нет примитивов в коде — `Int`, `Long`, но JVM-компилятор схлопывает их в `int`/`long`
>- нет checked exceptions
>- `==` это `.equals()`, `===` это сравнение ссылок

# [[KotlinCore]]
# [[KOTLIN-TYPES]]
# [[KOTLIN-COLLECTIONS]]
# Sequences ([[Kotlin Sequences]])  — аналог Stream API
# Поразрядные операции ([[Kotlin Bitwise Operators]])
# [[Kotlin JVM]]
# [[Kotlin Лямбда выражения]]
# [[Kotlin корутины]] — аналог потоков, асинхронность
# [[Kotlin JPA N+1 problem]]

# 👑 Итог — большая истина Kotlin

### 👉 null-safety встроена в систему типов — `String` ≠ `String?`.
### 👉 По умолчанию всё `final` и `public` — открывай явно через `open`/`internal`/`private`.
### 👉 `Any` — корень иерархии (аналог `Object`), но без `wait/notify/getClass`.
### 👉 Smart cast убирает необходимость явного `(String)x` после `is`.
### 👉 Data class даёт `equals/hashCode/toString/copy/componentN` из коробки.
### 👉 Под капотом — тот же JVM bytecode, что и Java.
