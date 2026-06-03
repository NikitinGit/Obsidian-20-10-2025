
# Памятка по HashMap, hashCode/equals и бакетам

> Примеры кода: `src/main/java/com/example/testlinux/java/core/hashcodetest/`
> (`_01_HowHashMapStores.java` ... `_09_BucketDistribution.java`)

---

## 1. Куда HashMap кладёт элемент

```
put(key, value):
  1) int h  = key.hashCode();          // прикладной хеш, любой int
  2) int hh = h ^ (h >>> 16);          // "spread" — размешиваем старшие биты к младшим
  3) int i  = hh & (table.length - 1); // индекс бакета (cap = степень двойки -> заменяет % cap)
```

- `table` — внутренний массив бакетов. По умолчанию длина 16, всегда степень двойки.
- `hashCode` НЕ обязан быть уникальным — он лишь говорит, **в каком бакете искать**.
- Равенство решает `equals`.
- Сам массив `table` **ленив**: до первого `put` он `null`.

### Пример формул
| `hashCode()` | spread | bucket (cap=16) |
|---|---|---|
| 0 | 0 | 0 |
| 1 | 1 | 1 |
| 16 | 16 | 0 |
| 17 | 17 | 1 |
| -1 | -65536 | 0 |
| Integer.MAX_VALUE | 2147450880 | 0 |

---

## 2. Коллизии

| Тип | Когда | Что делать |
|---|---|---|
| **Hash-коллизия** | одинаковый `hashCode()` | бакет один, узлы в цепочке |
| **Коллизия индекса** | разные `hashCode`, но один бакет (например h=1 и h=17 → `& 15 == 1`) | то же — попадают в один бакет |

### Что делает HashMap при коллизии (Java 8+)

1. В бакете лежит односвязный список `Node`.
2. При `put`: идём по узлам.
   - `n.hash == h && (n.key == key || n.key.equals(key))` → **перезапись value**.
   - иначе → добавить в начало бакета.
3. При `size > threshold` → **resize**.
4. При длине бакета ≥ **8** И `capacity ≥ 64` → **treeify** (красно-чёрное дерево).
5. При сжатии < **6** → дерево обратно в список.

### Поиск (`get`)
- Считаем бакет.
- Идём по списку/дереву.
- Ищем узел, у которого `hash == h` **И** `equals == true`.
- Не нашли → `null`.

**Главный принцип:** «попасть в бакет» ≠ «найти entry». Дальше всё равно `hash` + `equals`. Бакет — это лишь грубый адрес.

---

## 3. capacity, loadFactor, threshold, resize

| Параметр | По умолчанию | Что значит |
|---|---|---|
| capacity | 16 | длина `table` (бакетов) |
| loadFactor | 0.75f | максимальная плотность `size / capacity` |
| threshold | `capacity * loadFactor` = 12 | предел, после которого resize |

### Когда происходит resize
- `size > threshold` → capacity удваивается, **все элементы перераскладываются** (`hash & (newCap - 1)`).
- Это дорогая операция: O(n).

### Trajectories
- `new HashMap<>()` + 30 put: 16 → 32 → 64 (два resize)
- `new HashMap<>(64)` + 30 put: 64 (без resize)

### tableSizeFor: округление вверх до степени двойки
```
new HashMap<>(10)  -> capacity 16
new HashMap<>(17)  -> capacity 32
new HashMap<>(100) -> capacity 128
new HashMap<>(1000)-> capacity 1024
```

### Идиома «без resize вообще»
```java
new HashMap<>((int)(expectedSize / 0.75f) + 1)
// Java 19+:
HashMap.newHashMap(expectedSize)
```

### `new HashMap<>()` vs `new HashMap<>(16)`
- Поведенчески **идентично**: после первого put оба имеют capacity=16, threshold=12.
- Единственная разница — до первого put в поле `threshold`: 0 vs 16 (внутренний hint).

### Ленивая инициализация
| Момент | `table` | `threshold` |
|---|---|---|
| после `new HashMap<>()` | `null` | 0 |
| после `new HashMap<>(64)` | `null` | 64 (подсказка) |
| после 1-го put для `(64)` | массив длины 64 | 48 (`= 64 * 0.75`) |

---

## 4. Контракт equals / hashCode

1. `a.equals(b) == true` → `a.hashCode() == b.hashCode()`.
2. `a.hashCode() == b.hashCode()` НЕ влечёт равенство.
3. Пока объект в коллекции — его hash должен быть стабилен.

### Что ломается при нарушении

| Что не так | Симптом |
|---|---|
| `equals` есть, `hashCode` нет | дубликаты в `Set`, "потеря" ключа в `Map` |
| `hashCode` зависит от изменяемого поля | объект "теряется" после мутации |
| `hashCode` всегда константа | всё в один бакет, деградация до `O(n)` или `O(log n)` после treeify |
| `equals` слишком "ленивый" (всегда true) | разные ключи склеиваются в один |
| `equals` по неидентифицирующему полю | возврат чужого значения (см. `_08`) |

### Сценарий "ложно-положительный get"
Чтобы `get(незнакомый ключ)` вернул чужое значение, нужны **обе** поломки:
1. Совпавший `hashCode` (ступень 1).
2. `equals` врёт между разными по смыслу ключами (ступень 2).

```java
// Сломанный equals по hashValue:
@Override public int hashCode() { return hashValue; }
@Override public boolean equals(Object o) {
    return o instanceof BrokenEqualsKey k && hashValue == k.hashValue;
}

// 3 ключа в один бакет (hash 1, 17, 33 -> & 15 == 1), но id разные.
// put не считает их дубликатами (hash разный).
// get(gamma, hash=17) -> натыкается на beta (hash совпал), equals "обманывает" -> возврат beta.
```

---

## 5. HashSet = обёртка над HashMap

- Внутри `HashSet` лежит `private HashMap<E, Object> map`.
- value всегда константа `PRESENT = new Object()`.
- `add(e)` = `map.put(e, PRESENT) == null`.
- `contains(e)` = `map.containsKey(e)`.
- Всё, что про `HashMap`, относится к `HashSet` целиком.

`LinkedHashSet` / `LinkedHashMap` — двусвязный список поверх бакетов:
- порядок вставки сохраняется;
- access-order режим у `LinkedHashMap` позволяет реализовать **LRU-кэш**:

```java
new LinkedHashMap<>(16, 0.75f, true /* accessOrder */) {
    @Override protected boolean removeEldestEntry(Map.Entry eldest) {
        return size() > 3;
    }
}
```

---

## 6. Treeify (красно-чёрное дерево в бакете, Java 8+)

| Константа | Значение | Что значит |
|---|---|---|
| `TREEIFY_THRESHOLD` | 8 | при длине бакета ≥ 8 — попытка treeify |
| `MIN_TREEIFY_CAPACITY` | 64 | НО только если `table.length ≥ 64` |
| `UNTREEIFY_THRESHOLD` | 6 | при сжатии < 6 — дерево обратно в список |

Если `capacity < 64`, вместо treeify происходит **resize** (надеется, что после перераспределения коллизий станет меньше).

### Эффективность
- список: `O(k)` поиск
- дерево: `O(log k)` поиск
- ключи с `Comparable` дают лучшее дерево

### Бенчмарк (5000 get-ов)
```
обычные хеши:                   ~0.26 ms
коллизии + Comparable (дерево): ~2.4 ms
коллизии без Comparable:         ~38 ms   <- ×150 разница
```

Treeify — защита от **DoS-атак через подбор хешей**.

---

## 7. Распределение по бакетам и парадокс дней рождения

**Идеальный hash + small capacity всё равно даёт коллизии.**

Пример: `HashMap(16)` + 12 случайных Integer:
```
бакетов с 0 элементами: 6   ← 37% пустуют
бакетов с 1 элементом:  8
бакетов с 2 элементами: 2   ← коллизии при идеальном hash
```

### Парадокс дней рождения (асимптотика)
Для N случайных элементов в N бакетах:
- **~37% бакетов остаются пустыми** (→ `1/e`)
- **~26% бакетов имеют коллизии**
- Максимальная длина бакета растёт **очень медленно** (как `log N`)

Эти константы не зависят от N. Поэтому «1 элемент на бакет» **физически недостижимо** при разумной capacity.

### Что реально снижает коллизии
1. Больший `initialCapacity` (но память).
2. Меньший `loadFactor` (но память + частые resize).
3. Хороший `hashCode` (`Objects.hash(...)`, а не `id % 10` или константа).

---

## 8. Сложность операций

### Per-bucket
| Что в бакете | Стоимость |
|---|---|
| `null` или 1 узел | `O(1)` |
| Список из k узлов | `O(k)` |
| Дерево с k узлов | `O(log k)` |

### Для всей карты
| Операция | Best | Average | Worst (список) | Worst (дерево) |
|---|---|---|---|---|
| `get` / `put` / `remove` | `O(1)` | `O(1)` | `O(N)` | `O(log N)` |
| `containsValue` | `O(N)` | `O(N)` | `O(N)` | `O(N)` |

### Почему amortized O(1) работает
1. `loadFactor 0.75` → среднее ≤ 0.75 элемента на бакет.
2. `resize` при превышении → удвоение capacity.
3. `treeify` → даже патологический бакет → `O(log k)`.

**В норме k ≈ 1–3**, даже если N огромен. Без специальной поломки `O(N)` не достичь.

### Когда деградация реальна
- кастомный `hashCode = константа` или `id.length()`
- DoS-атака (подобранные ключи) — но treeify смягчает
- мутация полей, по которым считается hash → "теряется" в карте

---

## 9. equals + Hibernate proxy

### Проблема
```java
Event e = em.find(Event.class, 5);              // настоящий Event
Event p = em.getReference(Event.class, 5);      // прокси (Event$HibernateProxy$xyz)

e.getClass() != p.getClass();                   // true — разные классы
e.equals(p);                                    // должно быть true, но...
```

### Решение — `getPersistentClass()`
```java
Class<?> oEffectiveClass = o instanceof HibernateProxy
    ? ((HibernateProxy) o).getHibernateLazyInitializer().getPersistentClass()
    : o.getClass();
```

`LazyInitializer.getPersistentClass()` возвращает **реальный entity-класс** (`Event.class`), без триггера загрузки прокси.

Если прокси на наследника (`TournamentEvent extends Event`) — вернётся `TournamentEvent.class`.

---

## 10. Почему `@EqualsAndHashCode` на JPA-entity опасен

Lombok предупреждает: "severe performance and memory issues". Причины:

### 10.1 Изменчивый hash при persist
```java
Event e = new Event();           // id == null, hashCode == 0
set.add(e);                       // лёг в бакет 0
em.persist(e);                    // id := 42, hashCode == 42, должен быть в бакете 10
set.contains(e);                  // false — объект "потерян"
```

### 10.2 Ленивая загрузка коллекций
Дефолтный `@EqualsAndHashCode` сравнивает **все поля**, включая `@OneToMany`.
- `set.add(event)` → `hashCode` трогает коллекцию → SELECT в БД → N+1 на пустом месте.
- Вне сессии — `LazyInitializationException`.

### 10.3 Прокси
Lombok не различает прокси и реальный entity, использует `getClass()`.

### Рукописный паттерн (как в `Event.java`)
```java
@Override public final boolean equals(Object o) {
    if (this == o) return true;
    if (o == null) return false;
    Class<?> oCls = o instanceof HibernateProxy
        ? ((HibernateProxy) o).getHibernateLazyInitializer().getPersistentClass()
        : o.getClass();
    Class<?> thisCls = this instanceof HibernateProxy
        ? ((HibernateProxy) this).getHibernateLazyInitializer().getPersistentClass()
        : this.getClass();
    if (thisCls != oCls) return false;
    Event event = (Event) o;
    return getEventId() != null && Objects.equals(getEventId(), event.getEventId());
}

@Override public final int hashCode() {
    return this instanceof HibernateProxy
        ? ((HibernateProxy) this).getHibernateLazyInitializer().getPersistentClass().hashCode()
        : getClass().hashCode();
}
```

Особенности:
- `hashCode` = хеш класса → **стабилен всю жизнь** объекта.
- Все entity одного типа в одном бакете — сознательный размен на корректность.
- `equals` смотрит только на `id` (с `!= null` проверкой для unpersisted).
- Прокси и реальный entity корректно сравниваются через `getPersistentClass()`.

Альтернатива получше — **бизнес-ключ** (UUID/email), известный до persist:
```java
@Override public int hashCode() { return Objects.hashCode(businessKey); }
```

---

## 11. Pattern matching for instanceof (Java 16+)

Стандартный `equals` теперь пишут так:
```java
@Override public boolean equals(Object o) {
    return o instanceof Foo f && Objects.equals(id, f.id);
}
```

- `instanceof Foo f` сразу даёт переменную `f` типа `Foo`.
- `null` автоматически отсекается (`null instanceof X` всегда `false`).
- Каст руками не нужен.
- Работает в `&&` (правая часть видит биндинг), но не в `||`.

Эквивалент старого стиля:
```java
@Override public boolean equals(Object o) {
    if (!(o instanceof Foo)) return false;
    Foo f = (Foo) o;
    return Objects.equals(id, f.id);
}
```

---

## 12. Шпаргалка-резюме

- `HashMap` — массив бакетов; индекс = `(spread(hash)) & (cap - 1)`.
- `put`/`get` сначала проверяют `hash`, потом `equals` — это **двухступенчатый** фильтр.
- Бакет — лишь грубый адрес. Реальное «нашлось» решает `equals`.
- В бакете может быть **0, 1 узел, список или дерево** — это нормально и неизбежно.
- Контракт: `equals == true` → `hashCode == hashCode`. Нарушение даёт тихие баги.
- `equals` должен сравнивать **идентифицирующие** поля, `hashCode` — **по тем же** полям.
- Поля, по которым считается hash, **не должны меняться** пока объект в коллекции.
- Сложность HashMap — amortized `O(1)`, treeify даёт `O(log n)` в худшем случае.
- Для JPA-entity — рукописный `equals` с `HibernateProxy`, не `@EqualsAndHashCode`.
- Идеальный `equals` (современный): `o instanceof Foo f && Objects.equals(id, f.id)`.
- `HashSet`/`LinkedHashSet` — обёртки над `HashMap`/`LinkedHashMap`.

---

## 13. Перекрёстные ссылки

- `_01_HowHashMapStores.java` — формула `hash -> bucket`, коллизия по `hashCode`.
- `_02_CollisionsAndBuckets.java` — хеш-коллизия vs коллизия индекса, мини-бакет.
- `_03_ResizeAndLoadFactor.java` — capacity / threshold / resize-trajectories.
- `_04_EqualsHashContract.java` — что ломается при нарушении контракта.
- `_05_HashSetIsHashMap.java` — `HashSet`/`LinkedHashSet`/LRU.
- `_06_TreeifyDemo.java` — treeify, бенчмарк дерево vs список vs идеальный hash.
- `_07_StepByStepPut.java` — самописная MiniMap, путь `put` шаг за шагом.
- `_08_BrokenEqualsMultipleEntries.java` — ложно-положительный `get` через сломанный equals.
- `_09_BucketDistribution.java` — реальное распределение по бакетам + парадокс дней рождения.

См. также:
- [[JAVA-TYPES]] — типы, generics, касты.
- [[JPA N+1 problem]] — почему важна правильная стратегия загрузки.