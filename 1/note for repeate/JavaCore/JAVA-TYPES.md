
# Памятки по типам и кастам в Java

> Примеры кода: `src/main/java/com/example/testlinux/java/core/types/`
> (`ClassVsType.java`, `CastingKinds.java`, `ArrayTypes.java`, `GenericTypes.java`, `Variance.java`)

---

## 1. ClassCastException — почему возникает

```java
Animal an = new Animal();
Dog dog = (Dog) an;   // ClassCastException
```

- В строке 1 создаётся объект **именно класса `Animal`**.
- `Dog extends Animal` → каждый `Dog` — это `Animal`, но **не каждый `Animal` — это `Dog`**.
- JVM на инструкции `checkcast` сверяет фактический рантайм-тип и кидает `ClassCastException`.

### Как исправить

| Способ | Когда применять |
|---|---|
| `Animal an = new Dog();` | если реально нужен `Dog` |
| `if (an instanceof Dog d) { d.execute(); }` | безопасный каст через pattern matching (Java 16+) |
| `assertThrows(ClassCastException.class, …)` | если тест должен демонстрировать само исключение |

### Почему в `Event#equals` каст `(HibernateProxy) o` не падает

```java
Class<?> oEffectiveClass = o instanceof HibernateProxy
        ? ((HibernateProxy) o).getHibernateLazyInitializer().getPersistentClass()
        : o.getClass();
```

Каст стоит **внутри ветки `?` тернарного оператора, после `instanceof`**. Если `o` не `HibernateProxy`, JVM выполняет только ветку после `:`. Каст никогда не выполняется при несовпадении типа → CCE невозможен. Это **safe-cast паттерн**: проверка `instanceof` гарантирует валидность каста.

В `StreamTest#equalsTest` каст **безусловный** → `(HibernateProxy) new Object()` падает.

---

## 2. Класс vs Тип

**Класс ≠ Тип.**

- **Класс** — синтаксическая конструкция (`class Dog {}`), один `Class`-объект в runtime на classloader.
- **Тип** — категория системы типов компилятора. Типов **больше**, чем классов.

### Что есть тип, но не класс

- **Примитивы**: `int`, `long`, `double`, `boolean`, ...
- **Интерфейсы**: `Runnable`, `Comparable<T>`
- **Массивы**: `int[]`, `Dog[][]` — нет `.java` файла
- **Параметризованные**: `List<String>` — отдельный тип от `List<Integer>`
- **Wildcard**: `List<? extends Animal>`, `List<?>`
- **Type variables**: `T`, `E`
- **null-тип** — у литерала `null` собственный тип

### Может ли у двух классов быть один тип?

**Нет.** Каждый класс задаёт уникальный именованный тип. Даже одноимённые классы из разных пакетов — разные типы. Тот же класс, загруженный двумя `ClassLoader`'ами, — тоже два разных типа в runtime.

### Может ли у одного класса быть много типов?

**Да.**

```java
ArrayList<String>           // тип №1
ArrayList<Integer>          // тип №2
ArrayList                   // raw type
ArrayList<?>                // wildcard
ArrayList<? extends Animal> // bounded wildcard
```

Все это разные типы для компилятора, **но один класс в runtime** (type erasure):

```java
new ArrayList<String>().getClass() == new ArrayList<Integer>().getClass();  // true
```

### Несколько типов у одного объекта

```java
Dog d = new Dog();
Animal a = d;        // тот же объект, тип переменной — Animal
Object o = d;        // тип — Object
Runnable r = d;      // если Dog implements Runnable
```

---

## 3. Виды кастов

### 3.1 По направлению в иерархии

| Каст | Описание | Пример |
|---|---|---|
| **Upcast** | к супертипу, безопасен, обычно неявный | `Animal a = new Dog();` |
| **Downcast** | к подтипу, явный, runtime-проверка | `Dog d = (Dog) animal;` |
| **Sidecast** | между несвязанными интерфейсами | `Serializable s = (Serializable) runnable;` |

### 3.2 Примитивы

| Каст | Описание | Пример |
|---|---|---|
| **Widening** | расширение, без потерь, неявно | `long l = 10;` |
| **Narrowing** | сужение, явное, потери | `int i = (int) 3.99;` → `3` |

### 3.3 Боксинг

| Операция | Пример |
|---|---|
| **Boxing** | `Integer x = 5;` → `Integer.valueOf(5)` |
| **Unboxing** | `int i = x;` → `x.intValue()` (NPE при `null`) |

### 3.4 Прочие

- **Unchecked cast** в generics — `(List<String>) obj`, warning, в runtime проверяется только raw-тип.
- **`Class.cast(obj)`** — программный аналог `(T) obj`.
- **Pattern matching** (Java 16+): `if (o instanceof Dog d) { d.bark(); }` — безопасная альтернатива.

### Зачем `((Dog) an).methodOfDog()`

Когда переменная **объявлена типом родителя**, но нужно вызвать метод **только потомка**.

- В тривиальном `Animal an = new Dog()` каст избыточен — лучше сразу `Dog an = new Dog()`.
- В реальном коде переменная — `Animal` **не по выбору, а по контракту**:
  - элемент `List<Animal>`,
  - параметр метода `void handle(Animal a)`,
  - возврат фабрики `animalFactory.create(...)`,
  - `equals(Object o)` — сигнатуру диктует `Object`.

Если каст постоянный — **запах кода**, лучше:
- добавить виртуальный метод в родителя,
- использовать `sealed` + pattern matching switch.

---

## 4. Массивы как типы

- Массив — **полноценный ссылочный тип**, не класс в обычном смысле.
- У каждой размерности и element-типа — **свой `Class`-объект** (erasure здесь нет!).
- Все массивы — подтипы `Object`, `Cloneable`, `Serializable`.
- Имена JVM: `[I` = `int[]`, `[[I` = `int[][]`, `[Lpkg.Cls;` = `Cls[]`.

```java
int[].class      // class [I
int[][].class    // class [[I        ← другой Class
Dog[].class      // class [Lpkg.Dog;
Dog[].class != Animal[].class        // true
```

### Ковариантность массивов

```java
Dog[] dogs = { new Dog() };
Animal[] animals = dogs;          // Dog[] is-a Animal[] — OK
animals[0] = new Cat();           // ArrayStoreException в runtime
```

Расплата за ковариантность — runtime-проверка типа элемента. Generics этой проблемы избегают через инвариантность.

---

## 5. Generics-типы

### Один класс — много типов, но один `Class`

```java
new ArrayList<String>().getClass() == new ArrayList<Integer>().getClass();  // true
```

### Кирпичики

| Конструкция | Что значит |
|---|---|
| `List<String>` | параметризованный тип |
| `List` | raw type — обходит проверки, warning |
| `List<?>` | wildcard — какой-то конкретный, но неизвестный тип |
| `List<? extends Animal>` | bounded wildcard сверху |
| `List<? super Dog>` | bounded wildcard снизу |
| `<T extends Animal>` | type variable с границей |

### Из-за erasure нельзя

```java
o instanceof ArrayList<String>   // compile error
new T[10]                        // compile error
T.class                          // compile error
```

---

## 6. Вариативность (variance)

Пусть `Dog <: Animal`.

| Variance | Generics | Что разрешено |
|---|---|---|
| **Инвариантность** | `List<Dog>` ≠ `List<Animal>` | строгое равенство параметра |
| **Ковариантность** | `List<? extends Animal>` | принимает `List<Dog>`, `List<Cat>`, `List<Animal>`; **только чтение** |
| **Контравариантность** | `List<? super Dog>` | принимает `List<Dog>`, `List<Animal>`, `List<Object>`; **только запись Dog/подтипов** |

### Правило PECS

> **P**roducer **E**xtends, **C**onsumer **S**uper

- Коллекция **отдаёт** элементы → `? extends T`
- Коллекция **принимает** элементы → `? super T`

### Примеры

```java
// Ковариантность: читаем
void printAll(List<? extends Animal> animals) {
    for (Animal a : animals) System.out.println(a);   // OK
    // animals.add(new Dog());                        // compile error
}

// Контравариантность: пишем
void addDogs(List<? super Dog> sink) {
    sink.add(new Dog());                              // OK
    sink.add(new Puppy());                            // OK (Puppy <: Dog)
    // Animal a = sink.get(0);                        // compile error — только Object
    Object o = sink.get(0);
}
```

### Контравариантность функциональных интерфейсов

```java
Consumer<Animal> printAnimal = a -> System.out.println(a);
Consumer<? super Dog> dogConsumer = printAnimal;     // OK — Consumer контравариантен по T

Function<Animal, Dog> f = a -> new Dog();
Function<? super Dog, ? extends Animal> g = f;       // вход контравариантен, выход ковариантен

Comparator<Animal> byHash = Comparator.comparingInt(Object::hashCode);
List<Dog> dogs = ...;
dogs.sort(byHash);                                   // Comparator<? super Dog>
```

### Массивы vs Generics

| | Массивы | Generics |
|---|---|---|
| Вариативность | **ковариантны** (`Dog[] <: Animal[]`) | **инвариантны** (`List<Dog>` ≠ `List<Animal>`) |
| Проверка типа | runtime → `ArrayStoreException` | compile-time |
| Erasure | нет, у каждого типа свой `Class` | да, общий `Class` |
| Управление variance | автоматически | wildcards `? extends/super` |

---

## 7. Pattern matching switch — версии Java

| Java | Статус switch с pattern matching |
|---|---|
| 17 | **Preview** (1-я итерация), нужен `--enable-preview` |
| 18–20 | Preview, итерации 2–4 |
| **21** | **Стабильно**, работает без флагов |

```java
// Java 21+
switch (an) {
    case Dog d   -> d.methodOfDog();
    case Cat c   -> c.methodOfCat();
    case Snake s -> s.methodOfSnake();
    default      -> { }
}
```

В Java 17 без preview работает только `instanceof` pattern matching (стабильно с Java 16):

```java
if (an instanceof Dog d) d.methodOfDog();
else if (an instanceof Cat c) c.methodOfCat();
else if (an instanceof Snake s) s.methodOfSnake();
```

С `sealed` интерфейсом в Java 21 switch становится **исчерпывающим** — `default` не нужен:

```java
sealed interface Animal permits Dog, Cat, Snake {}

switch (an) {
    case Dog d   -> ...;
    case Cat c   -> ...;
    case Snake s -> ...;
    // default не нужен — компилятор видит все варианты
}
```

---

## 8. Шпаргалка-резюме

- **Каст падает** только когда фактический runtime-тип не совместим с целевым.
- **Безопасный downcast** — `instanceof` + pattern matching, либо `Class.cast`.
- **Класс — одно, тип — другое**: примитивы, интерфейсы, массивы, generics-параметризации — это типы без отдельного класса (или с одним общим классом на много типов).
- **Generics инвариантны**, **массивы ковариантны** (с runtime-проверкой).
- **PECS**: producer → `? extends T`, consumer → `? super T`.
- **`o instanceof Dog d`** — Java 16+. **Switch с pattern matching** — стабильно с Java 21.