# equals / hashCode в JPA-сущностях

## Зачем нужен ручной equals/hashCode

>[!question]- Почему дефолтный `Object.equals` (по ссылке) для JPA-сущности — плохо?
> Hibernate регулярно создаёт **несколько Java-объектов для одной строки БД**: после `entityManager.clear()`, между разными транзакциями/сессиями, через lazy-прокси, после `detach()` или сериализации в JSON. По ссылке (`==`) такие объекты НЕ равны, поэтому `HashSet.contains`, `HashMap.get`, `List.contains` и любой `a.equals(b)` теряют сущность — хотя по бизнес-смыслу это одна и та же запись.

>[!question]- Какие 3 типичных сценария ломаются БЕЗ ручного equals?
> 1. **Разные транзакции/сессии**: `findById(5)` в двух разных транзакциях → два разных Java-объекта, `Object.equals` → false.
> 2. **HibernateProxy lazy-связи**: `bid.getEvent()` (lazy) → прокси, `eventRepo.findById(...)` в другой сессии → реальный Event. Разные классы → false.
> 3. **Detach + reload** (или сериализация в JSON туда-обратно): `entityManager.detach(saved)` + `findById(savedId)` → новый Java-объект, `Object.equals` → false.

## Lombok @EqualsAndHashCode

>[!question]- Что не так с `@EqualsAndHashCode(of = "id")` от Lombok для JPA — какие 3 бага?
> 1. **Транзиентные сущности равны**: два `new Event()` (оба `id == null`) → `Objects.equals(null, null) == true` → в `HashSet` попадёт только одна.
> 2. **`hashCode` меняется после `persist()`**: до сохранения `id == null` (один хэш), после — id появился (другой хэш). Если положили в `HashSet` ДО save — после save `contains()` не найдёт, объект "потерян" в неправильном bucket.
> 3. **Не учитывает HibernateProxy**: lazy-связь возвращает прокси, у него другой `getClass()`. Lombok этого не разворачивает.

>[!question]- Что значит `@EqualsAndHashCode.Exclude` без класс-уровневой `@EqualsAndHashCode`?
> **Ничего.** `@Exclude`/`@Include` работают только если на классе стоит `@EqualsAndHashCode(...)`. Иначе Lombok их игнорирует, equals/hashCode не генерирует, используется реализация от `Object` (по ссылке).

>[!question]- Почему нельзя смешивать `@EqualsAndHashCode(of = "id")` и `@EqualsAndHashCode.Exclude` на полях?
> Lombok падает компиляцией с ошибкой:
> `The old-style 'exclude/of' parameter cannot be used together with the new-style @Include / @Exclude annotations.`
> Нужно выбрать один стиль: либо `of = "..."`/`exclude = "..."`, либо `onlyExplicitlyIncluded = true` + `@Include`/`@Exclude` на полях.

## Канонический ручной вариант

>[!question]- Как выглядит канонический ручной `equals` для JPA и почему так?
> ```java
> @Override
> public final boolean equals(Object o) {
>     if (this == o) return true;
>     if (o == null) return false;
>     Class<?> oEffectiveClass = o instanceof HibernateProxy
>         ? ((HibernateProxy) o).getHibernateLazyInitializer().getPersistentClass()
>         : o.getClass();
>     Class<?> thisEffectiveClass = this instanceof HibernateProxy
>         ? ((HibernateProxy) this).getHibernateLazyInitializer().getPersistentClass()
>         : this.getClass();
>     if (thisEffectiveClass != oEffectiveClass) return false;
>     Event event = (Event) o;
>     return getEventId() != null && Objects.equals(getEventId(), event.getEventId());
> }
> ```
> - `HibernateProxy`-проверка разворачивает прокси-класс в реальный entity-класс — без этого прокси и обычный entity никогда не были бы равны.
> - `getEventId() != null` отсекает транзиентные: два `new Event()` с `id == null` НЕ равны (только `this == o` для них работает).
> - Сравнение по id означает: "одна запись в БД ⇒ один логический объект, сколько бы Java-копий ни было".

>[!question]- Почему канонический `hashCode` возвращает хэш КЛАССА, а не id?
> ```java
> @Override
> public final int hashCode() {
>     return this instanceof HibernateProxy
>         ? ((HibernateProxy) this).getHibernateLazyInitializer().getPersistentClass().hashCode()
>         : getClass().hashCode();
> }
> ```
> - **Главное требование к `hashCode`**: значение не должно меняться, пока объект лежит в `HashSet`/`HashMap`. Если бы возвращали `Objects.hash(id)`, то при `persist()` (id меняется с null на число) хэш бы поменялся → объект "потерялся" в коллекции (баг №2 из Lombok).
> - Хэш класса — **константа на всю жизнь объекта**, контракт `equals/hashCode` соблюдён.
> - Цена: все `Event`-сущности падают в один bucket. Распределение плохое, но **корректность важнее**.

>[!question]- Почему в `equals` проверка `(o instanceof HibernateProxy)` важнее, чем `o.getClass()`?
> Если просто сравнивать `this.getClass() == o.getClass()`, то прокси (`Event$HibernateProxy$xxx`) и обычный `Event` будут считаться разными классами, хотя это одна и та же запись в БД. `instanceof HibernateProxy` + `getHibernateLazyInitializer().getPersistentClass()` разворачивает прокси в реальный entity-класс.

>[!question]- Почему методы `equals`/`hashCode` в каноническом варианте помечены `final`?
> Чтобы Hibernate-прокси (сгенерированный подкласс) не могли их переопределить и нечаянно нарушить контракт. Прокси всё равно делегирует `equals`/`hashCode` через `HibernateLazyInitializer`, поэтому `final` на этих двух методах прокси не ломает.

>[!question]- Будут ли равны Event из `new` и Event из репозитория (с ручным equals)?
> Только если **руками выставить тот же ненулевой `id`** у созданного: `manual.setEventId(loaded.getEventId())`. Тогда `equals` вернёт true (он сравнивает только id). Без `setEventId(...)` транзиент с `id == null` отсеется условием `getEventId() != null`.

## HibernateProxy

>[!question]- `HibernateProxy` — это класс или интерфейс?
> **Интерфейс** (`org.hibernate.proxy.HibernateProxy extends Serializable`). Маркерный, без логики.

>[!question]- Какова иерархия наследования сгенерированного прокси?
> Прокси — это сгенерированный в рантайме **подкласс entity**, который ДОПОЛНИТЕЛЬНО реализует `HibernateProxy`:
> ```
> class Event$HibernateProxy$xxxxx
>         extends Event              ← ваш entity-класс БАЗОВЫЙ
>         implements HibernateProxy  ← интерфейс
> ```
> Поэтому `proxy instanceof Event` тоже `true`, и `proxy instanceof HibernateProxy` тоже `true`.

>[!question]- Когда генерируются прокси-классы — при старте приложения?
> **Нет, лениво — при первом использовании.** Если в коде нет ни одной `@ManyToOne(LAZY)`/`@OneToOne(LAZY)` на `Event` и не вызывается `entityManager.getReference(Event.class, ...)` — прокси-класса для `Event` Hibernate так и не сгенерирует. Генерацию делает ByteBuddy прямо в JVM при первой надобности, класс кэшируется в `SessionFactory`.

>[!question]- Какие требования к entity, чтобы Hibernate смог сгенерировать прокси?
> - Класс **не `final`** (иначе нельзя унаследоваться).
> - Есть **публичный/protected конструктор без аргументов** (Lombok: `@NoArgsConstructor`).
> - Методы, которые прокси должен перехватывать, **не `final`**.
> - Сами `equals`/`hashCode` можно делать `final` — прокси всё равно делегирует их через `HibernateLazyInitializer`.

## Когда прокси, когда реальный объект

>[!question]- Каково главное правило: когда я получу прокси, а когда реальный объект?
> **Прокси появляется только там, где Hibernate решил ОТЛОЖИТЬ загрузку.** Если решил грузить прямо сейчас (find/JPQL/JOIN FETCH/eager) → реальный объект. Если решил грузить позже (`getReference` или lazy-связь) → прокси. Прокси — это **обещание сходить в БД потом**.

>[!question]- Какие источники ВСЕГДА дают реальный объект (не прокси)?
> | Источник | Почему |
> |---|---|
> | `new Event()` | вообще не от Hibernate |
> | `repository.findById(...)`, derived-методы | SELECT уже выполнился |
> | `entityManager.find(Class, id)` | то же самое |
> | Результат JPQL / HQL / native запроса | SELECT прямо сейчас |
> | Связь, загруженная через `JOIN FETCH` | SELECT их сразу подтянул |
> | `@ManyToOne` без `LAZY` (EAGER по умолчанию!) | загрузилось вместе с родителем |
> | Элементы коллекции `@OneToMany` (сама коллекция-обёртка `PersistentBag`/`PersistentSet` — другой разговор) | при инициализации коллекция читает всех сразу |
> | `entityManager.persist(new X())` | вы дали реальный объект, Hibernate его управляет |

>[!question]- Какие источники ВСЕГДА дают прокси?
> | Источник | Почему |
> |---|---|
> | `entityManager.getReference(Class, id)` | специально создаёт прокси без SELECT'а |
> | `@ManyToOne(fetch = LAZY) Foo foo` через `parent.getFoo()` | LAZY = "грузить потом" |
> | `@OneToOne(fetch = LAZY)` через геттер | то же самое |
> | `@OneToOne` non-optional + non-owning сторона | Hibernate не может ленить без прокси (нужно знать "связь вообще существует?") |

>[!question]- Почему lazy-связь иногда возвращает НЕ прокси, а реальный entity?
> Если целевой entity **уже лежит в persistence context** (identity map) — Hibernate возвращает managed-ссылку, прокси не создаётся. Прокси появляется только если entity ещё НЕ в PC.
> Пример:
> ```java
> Event e = eventRepo.findById(5).get();   // реальный Event в PC
> EventBidFighter bid = bidRepo.findById(1).get();
> Event lazy = bid.getEvent();             // НЕ прокси! Hibernate увидел Event 5 в PC и вернул ту же ссылку
> ```

>[!question]- Что вернёт повторный `find()` после `getReference()` — реальный объект или прокси?
> **Тот же прокси** (после инициализации). Если в PC уже лежит прокси, повторный find его НЕ заменит на реальный объект:
> ```java
> Event proxy = entityManager.getReference(Event.class, 5);    // прокси в PC
> Event same = entityManager.find(Event.class, 5);             // ТОТ ЖЕ прокси
> proxy == same;                                                // true
> ```
> Чтобы получить разные Java-копии одной записи — нужно очистить PC (`entityManager.clear()`) или использовать разные сессии.

>[!question]- Какие хелперы у Hibernate помогают работать с прокси?
> ```java
> import org.hibernate.Hibernate;
> 
> Hibernate.isInitialized(x);     // true/false — успели проинициализировать
> x instanceof HibernateProxy;    // true/false — это прокси-обёртка
> Hibernate.unproxy(x);           // возвращает реальный entity из-под прокси
> Hibernate.getClass(x);          // возвращает реальный entity-класс (Event.class), даже если x — прокси
> ```
> `Hibernate.getClass(x)` особенно удобно вместо `x.getClass()`, когда нужен entity-тип независимо от того, прокси перед вами или реальный объект.

## Persistence context и сессии

>[!question]- Что такое identity map в Hibernate?
> Гарантия "одна строка БД ⇒ один Java-объект в пределах одной сессии". Persistence context хранит мапу `{(EntityType, id) → managed instance}`. Все запросы внутри сессии возвращают одну и ту же ссылку для одной записи. После закрытия сессии / `clear()` / `detach()` — гарантия теряется.

>[!question]- Когда `find()`/`findById()` идёт в БД, а когда нет?
> - `find(Class, id)` и `repository.findById(id)` **сначала проверяют PC** (identity map).
> - Если запись с таким id уже в PC → возвращают managed-ссылку, **SQL не выполняется** (в логах никакого SELECT'а не будет).
> - Если нет → выполняют SELECT, создают новый Java-объект, кладут в PC, возвращают.
> Это и есть **first-level cache** Hibernate'а.

>[!question]- Когда `@Query` (JPQL или native) идёт в БД?
> **Всегда**, даже если запись уже в PC. После сбора ResultSet'а Hibernate для каждой строки проверяет PC:
> - Если entity с таким id **уже** в PC → данные из ResultSet **выкидываются**, возвращается существующий managed-объект.
> - Если нет → создаётся новый Java-объект из ResultSet, кладётся в PC.
> 
> Это касается:
> - JPQL: `@Query("SELECT e FROM Event e WHERE ...")`
> - Native: `@Query(value = "...", nativeQuery = true)`
> - `entityManager.createQuery(...)` / `createNativeQuery(...)`
> - Criteria API
> - Specifications

>[!question]- Чем JPQL и native запрос отличаются в работе с PC?
> **Ничем.** JPQL — это просто синтаксис запросов по entity-классам; Hibernate его **парсит и транслирует в SQL**, дальше путь идентичен native. Оба:
> 1. Всегда выполняют SQL (PC не пропускает запрос).
> 2. При сборке результата проверяют PC для каждого id и используют существующие managed-ссылки, если они есть.
> Разница JPQL vs native — только в том, **кто пишет SQL**: при JPQL пишет Hibernate, при native — вы.

>[!question]- Почему `find()` пропускает SQL, а запросы — нет?
> JPA-спецификация требует, чтобы запросы возвращали **актуальные данные из БД**. Если бы запросы шли через PC, можно было бы пропустить апдейты, сделанные параллельно (другой транзакцией, native SQL'ом, миграцией) и получить устаревший снимок. `find()` — особое исключение: это **id-lookup**, и любое обновление по id всё равно прошло бы через `merge`/`refresh`/`flush` в той же сессии, поэтому пропуск SQL безопасен.

>[!question]- Сводная таблица: когда Hibernate идёт в БД при разных способах загрузки
> | Способ | Идёт в БД, если запись в PC | Возвращает |
> |---|---|---|
> | `entityManager.find(Class, id)` | **Нет** | managed из PC |
> | `repository.findById(id)` | **Нет** | managed из PC |
> | JPQL `@Query("SELECT e FROM Event e WHERE ...")` | **Да** | managed из PC (данные ResultSet выкидываются) |
> | Native `@Query(value = "...", nativeQuery = true)` | **Да** | managed из PC (данные ResultSet выкидываются) |
> | `entityManager.createQuery(...)` / Criteria | **Да** | managed из PC |
> | `entityManager.getReference(Class, id)` | **Нет** | прокси (или существующая ссылка из PC) |

>[!question]- Что делает `spring.jpa.open-in-view=true` и почему он мешает демо с прокси?
> OpenSessionInView — фильтр, который **держит одну сессию Hibernate и один persistence context на весь HTTP-запрос**. Даже отдельные `@Transactional` / `TransactionTemplate.execute(...)` стартуют новые транзакции, но **EntityManager и PC общие**.
> Следствие: identity map работает на весь запрос → нельзя получить две разных Java-копии одной записи без явного `entityManager.clear()`. Поэтому в демо приходится чистить PC руками.

>[!question]- Как принудительно получить прокси для lazy-связи?
> 1. `entityManager.flush()` + `entityManager.clear()` — выкинуть всё из PC.
> 2. Загрузить родительский entity (например, `bidRepository.findById(...)`).
> 3. Вызвать геттер lazy-связи (`bid.getEvent()`) — Event ещё НЕ в PC → Hibernate создаст прокси.
> Альтернатива: `entityManager.getReference(Event.class, id)` создаёт прокси безусловно, без обращения к БД.

>[!question]- Чем `entityManager.getReference()` отличается от `find()`?
> - `find(Class, id)` — идёт в БД (SELECT), возвращает полностью загруженный entity. Если нет — `null`.
> - `getReference(Class, id)` — НЕ идёт в БД, возвращает прокси-заглушку. SELECT выполнится при первом обращении к полю, отличному от id. Если записи нет — `EntityNotFoundException` бросится позже, при инициализации прокси.

>[!question]- Что такое `org.hibernate.Hibernate.initialize(proxy)`?
> Принудительно заполняет прокси данными из БД (запрашивает SELECT). Нужно, если планируется потом обращаться к полям прокси за пределами сессии — без инициализации в detached-состоянии будет `LazyInitializationException`. После `initialize` класс прокси не меняется (`Event$HibernateProxy$xxx`), он остаётся прокси, просто его внутреннее состояние заполнено.

>[!question]- Что такое `LazyInitializationException` и когда возникает?
> Бросается, когда обращаемся к НЕинициализированному lazy-полю (прокси или ленивая коллекция) **за пределами сессии Hibernate** (после закрытия транзакции/EntityManager без OpenSessionInView). Решения: инициализировать в транзакции (`Hibernate.initialize`, тач геттера, fetch join), вернуть DTO, или включить OEMIV (компромисс).

## Граничные случаи

>[!question]- Что произойдёт с `set.contains(event)` после `event.setEventId(...)` если equals по id?
> При попадании в `HashSet` бакет определяется по `hashCode()` **на момент добавления**. Если `hashCode` зависит от id и id потом изменился → ищем в другом бакете → `contains` вернёт false, хотя объект физически в сете. **Это причина**, по которой канонический `hashCode` возвращает хэш класса, а не id.

>[!question]- Почему `(insertable=false, updatable=false)` на поле может оставить его `null` после save?
> Если у вас два маппинга на одну колонку — например, `private Integer eventId` (read-only) и `@JoinColumn(name="idEvent") private Event event` — Hibernate при `save` пишет колонку через связь `event` и **не обновляет** Java-поле `eventId` в созданном объекте. Чтобы получить id, либо берите его через `bid.getEvent().getId()`, либо после save сделайте `flush + clear + findById` (тогда SELECT прочитает обе версии).

>[!question]- Почему `Set<Event>` в `@OneToMany(mappedBy=...)` опаснее, чем `List<Event>`?
> `Set.add` использует `hashCode`/`equals`. Если они кривые (как при `@EqualsAndHashCode(of="id")`), сразу всплывают баги. С `List` тех же проблем нет — он не вызывает `equals` при добавлении. Но при `List.contains`/`indexOf` баги всплывут так же. Универсально безопасный путь — корректный `equals`/`hashCode`, не выбор коллекции.

## Резюме

>[!question]- Ради чего вообще нужен ручной equals/hashCode по id — одним абзацем?
> Hibernate сам регулярно создаёт несколько Java-объектов для одной строки БД: между транзакциями, через lazy-прокси, после `clear`/`detach`/сериализации. Чтобы такие объекты считались "одной и той же сущностью" в `Set`/`Map`/`equals`-проверках, нужен equals, который сравнивает по id и разворачивает HibernateProxy, и hashCode, который **не меняется** при `persist()`. Lombok-сокращения `@EqualsAndHashCode(of="id")` дают обе ошибки (транзиентные равны + меняющийся hashCode), а IDE-generated equals с `getClass()`-чекером ломается на прокси. Поэтому пишут руками по шаблону Vlad Mihalcea.