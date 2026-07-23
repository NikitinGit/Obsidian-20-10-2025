# Proxy Object — что это и где встречается в Java/Spring/Hibernate

1. [x] How work @EqualsAndHashCode(of = "eventId") - write method
2. [x] Check with @EqualsAndHashCode(of = "eventId") your methods

## Базовое понятие

>[!question]- Что такое прокси-объект — одной фразой
> **Прокси (proxy)** — подмена обычного объекта после которой методы этого обычного объекта выполняет прокси объект
> Это объект-заместитель, который снаружи выглядит как обычный объект целевого типа (имеет тот же интерфейс или наследует тот же класс), но **перехватывает все вызовы методов** и добавляет/изменяет поведение: вызывает реальный объект, открывает транзакцию, идёт в БД за данными, кэширует результат, логирует и т. д.
> Принцип:
> ```
> Клиент → [proxy] → реальный объект (target)
>            ↑
>            перехват: до/после/вместо вызова
> ```
> Это паттерн "Proxy" из GoF — один из самых используемых в реальной разработке, хотя обычно вы его не видите явно: Spring и Hibernate генерируют прокси сами.

>[!question]- Зачем вообще нужны прокси — какие задачи они решают
> 1. **Отложить тяжёлую работу** (lazy loading): не загружать данные из БД, пока их реально не запросили. Это Hibernate-прокси для `@ManyToOne(LAZY)` / `getReference()`.
> 2. **Cross-cutting concerns** (сквозная логика): обернуть метод транзакцией, безопасностью, логированием, кэшем — НЕ меняя код самого метода. Это Spring AOP (`@Transactional`, `@Async`, `@Cacheable`, `@PreAuthorize`).
> 3. **Контроль доступа** (security proxy): проверять права перед делегированием вызова.
> 4. **Удалённый вызов** (remote proxy): прокси локально, реальный объект — на другой машине (RMI, gRPC stubs).
> 5. **Виртуальный объект** (virtual proxy): тяжёлый ресурс (изображение, документ) создаётся только при первом обращении.
> Общий мотив: **разделить "что делает метод" и "что должно произойти вокруг вызова"**.

>[!question]- Чем прокси отличается от декоратора или адаптера
> Все три обёртывают объект и реализуют его интерфейс, но:
> - **Адаптер**: меняет интерфейс (`Foo` → `Bar`). Цель — совместимость.
> - **Декоратор**: тот же интерфейс, добавляет/расширяет поведение (можно несколько слоёв). Цель — гибкое наращивание функций.
> - **Прокси**: тот же интерфейс, **контролирует доступ** к объекту. Часто прокси создаёт/держит target сам (lazy), декоратор всегда оборачивает уже существующий объект.
> На практике границы размыты — `@Transactional`-обёртка от Spring технически прокси, но ведёт себя как декоратор.

>[!question]- Типы прокси-объектов: статический vs динамический (JDK vs CGLIB)
>
> ## Типы прокси-объектов
>
> ### 1. Статический прокси (Static Proxy)
> Создается **вручную разработчиком** на этапе написания кода.
>
> **Пример:**
> ```java
> // Интерфейс
> public interface UserService {
>     void saveUser(String name);
> }
>
> // Реальная реализация
> public class UserServiceImpl implements UserService {
>     @Override
>     public void saveUser(String name) {
>         System.out.println("Saving user: " + name);
>     }
> }
>
> // СТАТИЧЕСКИЙ ПРОКСИ - написан вручную!
> public class UserServiceProxy implements UserService {
>     private UserService target;
>
>     public UserServiceProxy(UserService target) {
>         this.target = target;
>     }
>
>     @Override
>     public void saveUser(String name) {
>         System.out.println("BEFORE: Logging...");
>         target.saveUser(name);  // Делегирование к реальному объекту
>         System.out.println("AFTER: User saved");
>     }
> }
>
> // Использование:
> UserService service = new UserServiceProxy(new UserServiceImpl());
> service.saveUser("Igor");
> // Вывод:
> // BEFORE: Logging...
> // Saving user: Igor
> // AFTER: User saved
> ```
>
> **Проблемы статического прокси:**
> - ❌ Нужно вручную писать класс-прокси для каждого интерфейса
> - ❌ Дублирование кода (если 100 сервисов - нужно 100 прокси)
> - ❌ Сложно поддерживать (изменение интерфейса → изменение прокси)
>
> ### 2. Динамический прокси (Dynamic Proxy)
> Создается **автоматически в runtime** (во время выполнения программы).
>
> В Java есть **два механизма** динамических прокси:
>
> #### 2.1. JDK Dynamic Proxy (стандартный механизм Java)
> Работает **только с интерфейсами**.
>
> **Пример:**
> ```java
> import java.lang.reflect.InvocationHandler;
> import java.lang.reflect.Method;
> import java.lang.reflect.Proxy;
>
> // Интерфейс и реализация
> public interface UserService {
>     void saveUser(String name);
> }
>
> public class UserServiceImpl implements UserService {
>     @Override
>     public void saveUser(String name) {
>         System.out.println("Saving user: " + name);
>     }
> }
>
> // InvocationHandler - обработчик вызовов методов
> public class LoggingHandler implements InvocationHandler {
>     private Object target;
>
>     public LoggingHandler(Object target) {
>         this.target = target;
>     }
>
>     @Override
>     public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
>         System.out.println("BEFORE: " + method.getName());
>         Object result = method.invoke(target, args);  // Вызов реального метода
>         System.out.println("AFTER: " + method.getName());
>         return result;
>     }
> }
>
> // Создание динамического прокси:
> public class Main {
>     public static void main(String[] args) {
>         UserService realService = new UserServiceImpl();
>
>         // СОЗДАЕМ ДИНАМИЧЕСКИЙ ПРОКСИ В RUNTIME!
>         UserService proxyService = (UserService) Proxy.newProxyInstance(
>             UserService.class.getClassLoader(),
>             new Class<?>[]{UserService.class},  // Интерфейсы для прокси
>             new LoggingHandler(realService)
>         );
>
>         proxyService.saveUser("Igor");
>
>         // Проверка типа
>         System.out.println(proxyService.getClass().getName());
>         // Вывод: com.sun.proxy.$Proxy0 ← Динамический класс!
>     }
> }
> ```
>
> **Ограничение JDK Dynamic Proxy:**
> ```java
> // ❌ НЕ РАБОТАЕТ - нет интерфейса!
> public class UserServiceImpl {  // Без интерфейса
>     public void saveUser(String name) {
>         System.out.println("Saving user: " + name);
>     }
> }
>
> // ОШИБКА при попытке создать прокси:
> UserServiceImpl proxy = (UserServiceImpl) Proxy.newProxyInstance(...);
> // IllegalArgumentException: UserServiceImpl is not an interface
> ```
>
> #### 2.2. CGLIB Dynamic Proxy (библиотека)
> Работает **с классами БЕЗ интерфейсов** (создает подкласс через наследование).
>
> **Пример:**
> ```java
> import org.springframework.cglib.proxy.Enhancer;
> import org.springframework.cglib.proxy.MethodInterceptor;
> import org.springframework.cglib.proxy.MethodProxy;
>
> // Класс БЕЗ интерфейса
> public class UserService {
>     public void saveUser(String name) {
>         System.out.println("Saving user: " + name);
>     }
> }
>
> // Interceptor для CGLIB
> public class LoggingInterceptor implements MethodInterceptor {
>     @Override
>     public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
>         System.out.println("BEFORE: " + method.getName());
>         Object result = proxy.invokeSuper(obj, args);  // Вызов родительского метода
>         System.out.println("AFTER: " + method.getName());
>         return result;
>     }
> }
>
> // Создание CGLIB прокси:
> public class Main {
>     public static void main(String[] args) {
>         Enhancer enhancer = new Enhancer();
>         enhancer.setSuperclass(UserService.class);  // Наследуемся от класса!
>         enhancer.setCallback(new LoggingInterceptor());
>
>         UserService proxy = (UserService) enhancer.create();
>         proxy.saveUser("Igor");
>
>         // Проверка типа
>         System.out.println(proxy.getClass().getName());
>         // Вывод: UserService$$EnhancerByCGLIB$$12345678 ← CGLIB создал подкласс!
>     }
> }
> ```
>
> **Как работает CGLIB:**
> ```
>            UserService (оригинальный класс)
>                     ↑
>                     | наследование
>                     |
>     UserService$$EnhancerByCGLIB$$12345678 (динамически созданный подкласс)
>
>     @Override
>     public void saveUser(String name) {
>         interceptor.intercept(...);  // Вызов interceptor'а
>         super.saveUser(name);         // Затем вызов оригинального метода
>     }
> ```
>
> ## Сравнение всех типов прокси
>
> | Критерий | Статический прокси | JDK Dynamic Proxy | CGLIB Dynamic Proxy |
> |----------|-------------------|-------------------|---------------------|
> | **Создание** | Вручную программистом | Автоматически в runtime | Автоматически в runtime |
> | **Требует интерфейс** | Нет | ✅ ДА (обязательно!) | ❌ НЕТ (работает с классами) |
> | **Механизм** | Обычный класс | `java.lang.reflect.Proxy` | Наследование (создает подкласс) |
> | **Производительность** | Быстрый | Средняя | Чуть медленнее (из-за наследования) |
> | **Когда создается** | Compile-time | Runtime | Runtime |
> | **Название класса** | YourProxy | com.sun.proxy.$Proxy0 | YourClass$$EnhancerByCGLIB$$... |
> | **Используется в Spring** | Редко | Если есть интерфейс | Если нет интерфейса (или @Configuration) |
>
> ## Что использует Spring?
>
> Spring **автоматически выбирает** между JDK и CGLIB прокси:
>
> ```java
> // Случай 1: Есть интерфейс → JDK Dynamic Proxy
> public interface UserService {
>     void saveUser(String name);
> }
>
> @Service
> public class UserServiceImpl implements UserService {
>     @Override
>     public void saveUser(String name) {
>         System.out.println("Saving user: " + name);
>     }
> }
>
> // Spring создаст: com.sun.proxy.$Proxy123
>
>
> // Случай 2: Нет интерфейса → CGLIB Proxy
> @Service
> public class UserService {  // БЕЗ интерфейса!
>     public void saveUser(String name) {
>         System.out.println("Saving user: " + name);
>     }
> }
>
> // Spring создаст: UserService$$EnhancerBySpringCGLIB$$12345678
>
>
> // Случай 3: @Configuration → ВСЕГДА CGLIB (даже если есть интерфейс)
> @Configuration
> public class AppConfig {
>     @Bean
>     public UserService userService() {
>         return new UserServiceImpl();
>     }
> }
>
> // Spring создаст: AppConfig$$EnhancerBySpringCGLIB$$12345678
> ```
>
> ## Spring Data JPA + Custom Repository - какой прокси?
>
> В случае с `JudgeRoundScoreRepository`:
>
> ```java
> public interface JudgeRoundScoreRepository
>     extends JpaRepository<JudgesRoundScore, Long>,
>             JudgeRoundScoreRepositoryCustom {
>     // ...
> }
> ```
>
> Spring использует **JDK Dynamic Proxy**, потому что:
> - ✅ Есть интерфейс `JudgeRoundScoreRepository`
> - ✅ Spring Data создает прокси через `java.lang.reflect.Proxy`
>
> **Проверка:**
> ```java
> @Autowired
> private JudgeRoundScoreRepository repository;
>
> public void check() {
>     System.out.println(repository.getClass().getName());
>     // Вывод: com.sun.proxy.$Proxy123 ← JDK Dynamic Proxy!
>
>     System.out.println(Proxy.isProxyClass(repository.getClass()));
>     // Вывод: true
> }
> ```
>
> ## Когда Spring использует CGLIB вместо JDK?
>
> ```java
> // 1. @Configuration класс - ВСЕГДА CGLIB
> @Configuration
> public class MyConfig {
>     @Bean
>     public UserService userService() {
>         return new UserServiceImpl();
>     }
> }
>
> // 2. @Transactional на классе без интерфейса
> @Service
> @Transactional
> public class UserService {  // Нет интерфейса
>     public void saveUser(String name) { ... }
> }
>
> // 3. Принудительно через настройку
> @Configuration
> @EnableAspectJAutoProxy(proxyTargetClass = true)  // ← Форсировать CGLIB
> public class AppConfig { }
> ```
>
> ## Итого
>
> **Статический прокси:**
> - Пишется вручную
> - Используется редко (только для учебных целей или очень специфичных случаев)
>
> **Динамический прокси:**
> - **JDK Dynamic Proxy** (стандарт Java):
>   - Работает ТОЛЬКО с интерфейсами
>   - Использует `java.lang.reflect.Proxy`
>   - Быстрее CGLIB
>   - Используется Spring по умолчанию (если есть интерфейс)
>   - Название класса: `com.sun.proxy.$ProxyXXX`
>
> - **CGLIB Dynamic Proxy** (библиотека):
>   - Работает с классами БЕЗ интерфейсов
>   - Создает подкласс через наследование
>   - Используется Spring для @Configuration и классов без интерфейсов
>   - Название класса: `YourClass$$EnhancerBySpringCGLIB$$XXX`
>
> **В случае Spring Data JPA:**
> - Spring создает **JDK Dynamic Proxy** (`com.sun.proxy.$ProxyXXX`)
> - Прокси делегирует вызовы:
>   - В `SimpleJpaRepository` для JPA методов
>   - В `JudgeRoundScoreRepositoryCustomImpl` для кастомных методов

>[!question]- в GCLIB тоже все прокси динамические ?
>Да - код  создается автоматически прямо в оперативной памяти во время работы программы (в рантайме).

>[!question]- когда генерируется класс $Proxy147 в dynamic proxy  
>при внедрении зависимости через newProxyInstance после сканирования проекта ( всех бинов)  - не при расширении интерфейса extends JpaRepository<Event, Integer>
>// Примерно так Spring генерирует класс в памяти прямо во время старта:
>```
>// Примерно так Spring генерирует класс в памяти прямо во время старта:
>EventRepository repositoryInstance = (EventRepository) Proxy.newProxyInstance(
> EventRepository.class.getClassLoader(),
>   new Class<?>[]{EventRepository.class},
>   new JdkDynamicAopProxy( ... ) // передается созданный хэндлер
>   );
>```

>[!question]- жизненный цикл бина 
>- **Сканирование (Парсинг BeanDefinition):** Spring ищет аннотации (`@Component`, `@Service`, `@Repository`) и создает «чертежи» (инструкции) для создания бинов. Самих объектов в памяти еще нет.
>- - **Создание и Внедрение (Instantiation & DI):**
>     - Spring берет чертеж бина `OrderService`.
>     - Создает его **«сырой»** (оригинальный) экземпляр в памяти через конструктор.
>     - **Сразу же внедряет** в него зависимости (через `@Autowired` или конструктор), следуя паттерну Синглтон (ищет уже готовые бины в
>     - своем хранилище).
> - Инициализация и Проксирование (Initialization / BeanPostProcessor):
> -  Только _после_ того, как «сырой» объект создан и в него внедрили зависимости, вызываются методы инициализации (например, `@PostConstruct`).
> - **Самый финал:** Специальный обработчик (`BeanPostProcessor`) проверяет, нужны ли бину транзакции или кэш. Если нужны, он вызывает `Proxy.newProxyInstance()` для JDK Dynamic Proxy.
> - 1. - **Важно:** Полученный прокси-объект **заменяет** собой оригинальный сырой объект в хранилище Spring (IoC-контейнере). Все последующие классы, которые будут запрашивать этот бин, получат уже этот прокси.

>[!question]- отличие JDK Dynamic Proxy от CGLIB 
>1. JDK Dynamic Proxy генерирует прокси только для классов тех классов, которые реализуют интерфейс, а CGLIB  для всех классов
>2. CGLIB для инициализации прокси использует метод Enhancer.setSuperclass enhancer.create(), а JDK Dynamic Proxy  - Proxy.newProxyInstance() 
## Как создаются прокси в JVM

>[!question]- Прокси — это что-то особенное в JVM или обычный класс
> **Это обычный класс**, который не существует на диске и не написан вами. Его **байткод генерируется в рантайме** (при старте приложения или лениво при первом использовании) и сразу загружается в JVM. Для JVM это такой же класс, как любой другой; вы можете получить его `Class<?>`, вызвать методы, увидеть в стектрейсе.
> Примеры имён сгенерированных классов:
> - `com.sun.proxy.$Proxy42` — JDK dynamic proxy
> - `Event$$EnhancerBySpringCGLIB$$abc123` — CGLIB (Spring AOP)
> - `Event$HibernateProxy$7sioKrtU` — Hibernate ByteBuddy proxy

>[!question]- Какие 3 механизма генерации прокси-классов в Java мире
> 1. **JDK Dynamic Proxy** (`java.lang.reflect.Proxy`) — встроен в JDK с 1.3. Может проксировать **только интерфейсы** (target должен реализовывать хотя бы один). Прокси-класс наследует `java.lang.reflect.Proxy` и реализует те же интерфейсы. Все вызовы идут через ваш `InvocationHandler`.
> 2. **CGLIB** (Code Generation Library) — внешняя библиотека (вшита в `spring-core`). Может проксировать **классы напрямую** через наследование. Ограничения: класс не `final`, метод не `final`, нужен конструктор без аргументов (в новых версиях Spring — не обязательно).
> 3. **ByteBuddy** — более современная замена CGLIB. Используется Hibernate 5.x+ (раньше был CGLIB и Javassist). Тоже генерирует подклассы; API удобнее.
> Spring AOP по умолчанию выбирает: если у бина есть интерфейс — JDK proxy, иначе CGLIB. Через `@EnableAspectJAutoProxy(proxyTargetClass = true)` принудительно CGLIB.

>[!question]- Что означает запись вида `com.example.testlinux.domain.Event$HibernateProxy$7sioKrtU`
> Это имя **сгенерированного в рантайме подкласса** вашего `Event`, который Hibernate создал через ByteBuddy для lazy-связи или `getReference()`.
> Разбор по частям:
> - `com.example.testlinux.domain.Event` — пакет и имя родительского entity-класса (прокси наследует `Event`).
> - `$HibernateProxy$` — суффикс-маркер, добавляемый ByteBuddy при генерации прокси Hibernate'ом (раньше было `_$$_javassist_…`, в CGLIB-эпохе — `$$EnhancerByCGLIB$$…`).
> - `7sioKrtU` — **случайный хэш-суффикс**, чтобы имя класса было уникальным в пределах ClassLoader'а (можно сгенерировать тысячу прокси на один и тот же entity в разных сценариях; имена не должны конфликтовать).
> Иерархия классов:
> ```
> Event$HibernateProxy$7sioKrtU
>     extends Event                ← ваш entity
>     implements HibernateProxy    ← маркер Hibernate
> ```
> Поэтому `proxy instanceof Event == true` и `proxy instanceof HibernateProxy == true`.

>[!question]- Почему в имени прокси-класса есть `$$` или `$`
> Это **валидный, но необычный для рукописного кода символ в Java-имени класса** (в исходниках `$` обычно появляется только во вложенных классах: `Outer$Inner`). Генераторы прокси (CGLIB, ByteBuddy, JDK) специально используют двойной/одиночный `$` в имени, чтобы:
> - имена не конфликтовали с пользовательскими классами,
> - можно было визуально отличить сгенерированный байткод в стектрейсах и логах,
> - JVM-инструменты (JFR, debugger, profiler) могли отфильтровать "не пользовательский" код.

## Прокси в Spring AOP

>[!question]- Как Spring оборачивает бины в прокси и когда это происходит
> На этапе **создания бина** (`BeanPostProcessor` → `AbstractAutoProxyCreator`) Spring смотрит, есть ли у класса аннотации, которые требуют AOP-обёртки (`@Transactional`, `@Async`, `@Cacheable`, `@PreAuthorize`, кастомные `@Aspect` с pointcut'ом и т. д.). Если да — вместо самого бина в контейнер кладётся **прокси**, который оборачивает реальный экземпляр.
> Когда другой бин делает `@Autowired UserService userService` — на самом деле инжектится прокси. Реальный объект сидит внутри прокси и недоступен напрямую (получить можно через `AopProxyUtils.getSingletonTarget(bean)`).

>[!question]- Что делает прокси `@Transactional` при вызове метода
> Псевдокод обёртки:
> ```java
> public Result save(Entity e) {
>     TransactionStatus tx = transactionManager.getTransaction(...);
>     try {
>         Result r = target.save(e);   // вызов реального метода
>         transactionManager.commit(tx);
>         return r;
>     } catch (RuntimeException ex) {
>         transactionManager.rollback(tx);
>         throw ex;
>     }
> }
> ```
> То есть прокси открывает транзакцию ДО вызова реального метода, делает commit/rollback ПОСЛЕ. Сам метод о транзакции ничего не знает — это и есть "разделение бизнес-логики и сквозной".

>[!question]- Что такое self-invocation problem (вызов метода `this.save()` изнутри того же класса)
> Если внутри `UserService` есть два метода:
> ```java
> public void doStuff() {
>     this.save(...);   // ← НЕ через прокси
> }
> @Transactional
> public void save(...) { ... }
> ```
> Вызов `this.save(...)` идёт **напрямую к реальному объекту**, минуя прокси → AOP-аспект `@Transactional` НЕ срабатывает, транзакция не откроется. Это самая частая "потеря транзакции" в Spring-проектах.
> Обходы:
> 1. Вынести `save` в другой бин.
> 2. Инжектить себя через `ApplicationContext.getBean(...)` или `@Lazy` self-injection.
> 3. Использовать AspectJ load-time weaving (плетение прямо в байткод класса, без прокси) — тогда self-invocation тоже работает.

>[!question]- Почему `@Transactional` не работает на `private` или `final` методах
> Прокси перехватывает вызовы только через **публичный/protected виртуальный метод** (наследование):
> - `private` методы не наследуются → прокси не может их переопределить → перехвата нет.
> - `final` методы нельзя переопределить → CGLIB не может вставить hook.
> Решение для `final`: сделать non-final или переключиться на AspectJ. Решение для `private`: вынести в другой бин или сделать публичным.

>[!question]- Чем JDK proxy отличается от CGLIB proxy в Spring
> | Аспект | JDK Dynamic Proxy | CGLIB |
> |---|---|---|
> | Требование к target | Должен реализовывать интерфейс | Просто класс (не `final`) |
> | Механизм | Прокси реализует те же интерфейсы | Прокси наследует класс |
> | Что можно перехватывать | Только методы интерфейса | Любые non-final методы |
> | Зависимости | Только JDK | spring-core (тащит CGLIB) |
> | Скорость генерации | Быстрее | Медленнее (генерация подклассов сложнее) |
> | Поведение | `target instanceof Interface` — true | `target instanceof Class` — true |
> Spring выбирает автоматически (есть интерфейс → JDK), но обычно явно выставляют `proxyTargetClass=true` ради единообразия и работы с классами без интерфейсов.

## Прокси в Hibernate

>[!question]- Что такое PC (Persistence Context) — и почему без него ничего не понятно про Hibernate-прокси
> **PC = Persistence Context** (контекст персистентности) — это «рабочее пространство» Hibernate-сессии, внутренняя мапа `(EntityClass, id) → managed instance`, которую Hibernate ведёт для каждой сессии.
> Пока сущность лежит в PC — она **managed**:
> - Hibernate отслеживает её изменения (dirty checking).
> - При `flush()` синхронизирует с БД (SQL UPDATE).
> - При повторном `find()`/`findById()` возвращает **ту же Java-ссылку** (это и есть identity map / first-level cache).
> 
> Связь с прокси прямая: Hibernate создаёт прокси на entity **только если этого entity нет в PC**. Если уже лежит — отдаст managed-ссылку, прокси не нужен. Поэтому в demo-методах пишут `flush() + clear()` — чтобы выкинуть managed-копию из PC и заставить Hibernate создать прокси при следующем обращении.

>[!question]- Что какая операция делает с PC — таблица
> | Операция | Что происходит с PC |
> |---|---|
> | `entityManager.persist(e)` | Кладёт `e` в PC как managed |
> | `repository.save(e)` | То же самое (внутри `merge` или `persist`) |
> | `entityManager.find(...)` / `findById` | Сначала смотрит в PC; если нет — SELECT и кладёт в PC |
> | `entityManager.getReference(...)` | Кладёт **прокси** в PC без SELECT'а |
> | JPQL/native `@Query` | Всегда SELECT, но при сборке результата сверяется с PC: managed-объекты переиспользуются |
> | `entityManager.flush()` | Отправляет накопленные изменения managed-сущностей в БД. **PC не очищается.** |
> | `entityManager.clear()` | **Выбрасывает ВСЕ** сущности из PC — они становятся detached |
> | `entityManager.detach(e)` | Выбрасывает **одну конкретную** сущность из PC |
> | `entityManager.merge(e)` | Копирует состояние detached `e` в managed-объект из PC (или создаёт его) |
> | Завершение `@Transactional`-метода | По умолчанию `flush + close` сессии → PC уничтожается |

>[!question]- А зачем вообще вызывать `entityManager.clear()` или `detach()` — в обычной бизнес-логике это нужно
> **Почти никогда не нужно.** В обычном CRUD-сервисе их не пишут. Hibernate сам управляет PC через границы транзакции:
> ```
> [start tx] → load entity → modify → flush + commit → [end tx, PC уничтожен]
> ```
> Identity map внутри транзакции уже гарантирует "одна строка БД ⇒ одна Java-ссылка". В конце `@Transactional`-метода Hibernate сам делает `flush + close`, PC исчезает целиком. Очищать его вручную не нужно.
> Если `clear()` появляется в обычном сервисе — это **красный флаг**: что-то спроектировано неудачно (слишком длинная транзакция, возвращается entity вместо DTO, неправильный fetch-план и т. п.).

>[!question]- Тогда где `clear()`/`detach()` действительно нужны в продакшене
> Реальных кейсов мало:
> 1. **Batch processing — главный и почти единственный практический кейс.** Импорт миллиона строк:
>    ```java
>    for (int i = 0; i < records.size(); i++) {
>        entityManager.persist(new Customer(records.get(i)));
>        if (i % 50 == 0) {
>            entityManager.flush();   // отправить SQL батчем
>            entityManager.clear();   // ОЧИСТИТЬ PC, иначе OOM
>        }
>    }
>    ```
>    Без `clear()` все сохранённые сущности копятся в PC → гигабайты памяти + dirty checking за O(N) на каждом flush → `OutOfMemoryError`.
> 2. **Принудительная перечитка из БД** — когда другая транзакция (триггер, миграция, отдельный сервис на той же БД) изменила запись, и нужно гарантированно увидеть свежее состояние. Обычно решается через `entityManager.refresh(entity)` для одной сущности или через корректную транзакционную изоляцию, а не глобальным `clear()`.
> 3. **Detach перед сериализацией в JSON** (устаревший паттерн). Если контроллер возвращает entity напрямую — `detach` гасит dirty checking и lazy-инициализацию при сериализации. **Правильное решение — возвращать DTO**, а не entity, тогда `detach` не нужен.
> 4. **Тесты и обучающие demo.** Чтобы сымитировать "разные сессии" внутри одного `@Transactional`-метода и показать, как ведут себя `equals`/`hashCode`, прокси и identity map. Именно для этого `clear()` используется в `EventServiceTest#demonstrate*` методах — в production этого костыля не было бы.

>[!question]- Почему в обычной бизнес-логике беспокоиться о "двух представлениях одной строки" не нужно
> Hibernate **сам** обеспечивает то, о чём интуиция беспокоится:
> - **Атомарность** — `@Transactional` (commit/rollback).
> - **Консистентность "одна строка БД ⇒ одна Java-ссылка"** — identity map в пределах сессии.
> - **Освобождение памяти** — закрытие сессии в конце транзакции (PC исчезает целиком).
> Проблема "двух представлений" возникает **только на границах сессии**: разные HTTP-запросы, отдельные `@Transactional`-методы, сериализация в JSON туда-обратно, после `clear`/`detach`. Внутри одной транзакции этого не бывает. Поэтому в одной HTTP-операции `clear()` обычно вреден (теряете dirty checking, теряете managed-связи), а не полезен.

>[!question]- Если в коде встретился `entityManager.clear()` — какие проектные ошибки за этим обычно стоят
> | Симптом | Корень проблемы | Правильное решение |
> |---|---|---|
> | "После save кэш протух, нужно reload" | Слишком длинная транзакция, конкурентная запись | Сократить транзакцию, использовать `@Version` (optimistic locking) |
> | "Lazy-связь возвращает не то, что ожидал" | Неправильный fetch-план | `JOIN FETCH` / `@EntityGraph` ([[JPA N+1 problem]]) |
> | "Хочу избежать ненужного UPDATE при выходе из транзакции" | Возвращаете entity вместо DTO, dirty checking фиксит "грязное" поле | Возвращать DTO |
> | "PC растёт в долгом методе, OOM" | Слишком длинный `@Transactional` / batch без батчей | Разбить на батчи с `flush + clear` каждые N итераций (тот самый кейс №1) |
> | "Хочу принудительно перечитать запись" | Доверяете локальному PC больше, чем стоит | `entityManager.refresh(entity)` для одной записи, или правильная изоляция транзакций |

>[!question]- Сокращения и термины, которые часто встречаются вокруг PC
> | Аббревиатура | Расшифровка | Что это |
> |---|---|---|
> | **PC** | Persistence Context | Внутренняя мапа managed-сущностей одной сессии |
> | **EM** | EntityManager | JPA-API для работы с PC (`persist`, `find`, `flush`, `clear`...) |
> | **SF** | SessionFactory | Фабрика сессий Hibernate (одна на всё приложение) |
> | **L1 cache** | First-Level Cache | Синоним PC — это и есть кэш первого уровня Hibernate |
> | **L2 cache** | Second-Level Cache | Общий кэш на всю SessionFactory, между сессиями (опциональный: Ehcache, Caffeine, Hazelcast) |
> | **identity map** | — | Гарантия "одна строка БД ⇒ одна Java-ссылка в пределах сессии". Реализуется через PC. |
> | **managed** | — | Сущность лежит в PC, отслеживается Hibernate'ом |
> | **detached** | — | Сущность была managed, но выкинута из PC (`clear`, `detach`, конец сессии) |
> | **transient** | — | Только что созданная через `new`, в PC ни разу не была, id обычно `null` |
> | **removed** | — | `entityManager.remove(e)` вызван, но `flush` ещё не прошёл — на пути к удалению |

>[!question]- Чем Hibernate-прокси отличается от Spring-AOP-прокси
> Цель и поведение разные:
> - **Spring AOP proxy** оборачивает **бин-сервис**, чтобы добавить cross-cutting логику (транзакция, кэш, секьюрити). Target — обычный экземпляр сервиса, прокси про доступ.
> - **Hibernate proxy** оборачивает **entity-сущность**, чтобы отложить SELECT в БД. Target создаётся **только при первом обращении** к полю — до этого внутри прокси хранится только id. Прокси про загрузку.
> Технически: оба — это сгенерированные подклассы; различаются только по тому, какой `MethodInterceptor` подставлен.

>[!question]- Что такое `HibernateLazyInitializer` и почему он важен
> Объект, который **живёт внутри каждого Hibernate-прокси** и хранит:
> - id сущности (известен с момента создания прокси),
> - ссылку на сессию Hibernate (нужна, чтобы выполнить SELECT при инициализации),
> - флаг "уже инициализирован?",
> - реальный класс entity (для разворачивания через `getPersistentClass()`).
> Каждый перехваченный вызов метода прокси (кроме `getId()`) проходит через `HibernateLazyInitializer.initialize()` — если ещё не инициализирован, выполняется SELECT, данные кладутся в поля прокси, дальше вызов делегируется как обычно.
> Это то, что разворачивают канонические `equals`/`hashCode` для JPA — см. [[JPA equals hashCode]].

>[!question]- Где конкретно в коде возникает Hibernate-прокси
> | Триггер | Что получаем |
> |---|---|
> | `entityManager.getReference(Event.class, 5)` | Прокси без SELECT'а |
> | `bid.getEvent()` при `@ManyToOne(fetch = LAZY)` | Прокси на Event (если его нет в PC) |
> | `parent.getChild()` при `@OneToOne(fetch = LAZY)` non-optional | Прокси на Child |
> | Сериализация detached entity, у которого были lazy-связи | Прокси (но без сессии — обращение → `LazyInitializationException`) |
> Реальный entity (НЕ прокси) приходит из `find`/`findById`/JPQL/JOIN FETCH/native query/`new`.

>[!question]- Можно ли отключить Hibernate-прокси
> Можно — через `@ManyToOne(fetch = EAGER)` или `JOIN FETCH` в запросе — тогда связь грузится сразу реальным entity. Но это **обычно хуже**: EAGER тащит связь во ВСЕХ запросах, давая N+1 и `MultipleBagFetchException` на коллекциях. Лучше держать lazy + контролировать загрузку JOIN FETCH'ем там, где нужно (см. [[JPA N+1 problem]]).
> Также есть `@Proxy(lazy = false)` на entity-классе (deprecated в Hibernate 6) — отключает прокси-генерацию для класса вообще. Почти никогда не нужно.

>[!question]- Почему lazy-связь `bid.getEvent()` иногда возвращает НЕ прокси, а реальный Event — эксперимент
> Hibernate создаёт прокси **только если целевой entity ещё НЕ в persistence context**. Если Event уже в PC (identity map) — `bid.getEvent()` вернёт ту же managed-ссылку, прокси не нужен.
> Проверяется напрямую в `EventServiceTest#demonstrateLazyAssociationProxyProblem` — достаточно закомментировать `entityManager.clear()`:
> ```java
> Event savedEvent = eventRepository.save(e);   // Event лежит в PC
> // ...
> entityManager.flush();
> // entityManager.clear();   ← ЗАКОММЕНТИРОВАЛИ
>
> EventBidFighter loadedBid = bidRepository.findById(bidId).get();
> Event lazyEvent = loadedBid.getEvent();
> ```
> Вывод изменится:
> ```
> С    entityManager.clear():  lazyEvent instanceof HibernateProxy : true   ← прокси
> БЕЗ  entityManager.clear():  lazyEvent instanceof HibernateProxy : false  ← обычный Event
>                              lazyEvent.getClass() == Event.class  : true
> ```
> Что произошло: `save(e)` положил `savedEvent` в PC (`(Event, id=42) → savedEvent`). Без `clear()` эта запись там осталась. Когда `loadedBid.getEvent()` дёрнул lazy-связь, Hibernate сначала заглянул в PC, увидел Event с тем же id — и вернул ту же ссылку, **минуя создание прокси и без SELECT'а**.
> С `clear()` PC пуст → Hibernate вынужден создать прокси-заглушку (или сходить в БД при первом обращении к полю).
> Это и есть identity map в действии: "одна строка БД ⇒ один Java-объект в пределах сессии".

>[!question]- Что это значит на практике — когда в проекте я увижу прокси, а когда нет
> Зависит от **порядка операций в одной сессии**:
> - Если родитель и потенциальный таргет lazy-связи **загружаются в одной транзакции**, и таргет уже успел попасть в PC (через `save`, `find`, JPQL и т. д.) — связь вернёт реальный entity. Прокси не возникнет.
> - Если таргет НЕ в PC (новая сессия, после `clear`/`detach`, либо его никто не грузил) — связь вернёт прокси.
> Из-за этого один и тот же код `bid.getEvent()` может в одних сценариях отдавать прокси, в других — нет. Именно поэтому канонический `equals`/`hashCode` обязан корректно работать в обоих случаях: разворачивать прокси через `HibernateProxy.getHibernateLazyInitializer().getPersistentClass()` и при этом не ломаться на обычном entity.

>[!question]- А если НЕ вызывать `entityManager.clear()` — `getReference` создаст прокси
> **Нет, не создаст.** `getReference` тоже **сначала проверяет PC**. Если для запрашиваемого id там уже лежит managed-объект (реальный или прокси) — вернёт его. Прокси создаётся **только когда в PC ничего нет**.
> Пример из `demonstrateGetReferenceProblem`, если убрать `entityManager.clear()`:
> ```java
> Event saved = eventRepository.save(e);   // real Event в PC
> entityManager.flush();
> // entityManager.clear();    ← ЗАКОММЕНТИРОВАЛИ
> Event ref = entityManager.getReference(Event.class, saved.getEventId());
> 
> ref instanceof HibernateProxy;   // false — реальный Event
> ref.getClass() == Event.class;   // true
> ref == saved;                    // true — та же Java-ссылка из PC
> ```
> То есть в реальном коде проблемы "вдруг прокси" не будет, пока в этой же сессии родительский entity уже был тронут (save / find / любой запрос, поднявший его в PC). Прокси заводится **только на границах сессии** или после явного `clear`/`detach`.

>[!question]- Общее правило: когда я получу прокси, а когда реальный объект — одной формулой
> **PC — хозяин ситуации.** Hibernate ВСЕГДА сначала смотрит в PC, что бы вы ни вызвали:
> ```
>            Есть entity с этим id в PC?
>                     ↓
>           ┌─────────┴─────────┐
>         нет                  да
>          ↓                    ↓
>   действовать по         вернуть то,
>   логике вызова          что лежит в PC
>                          (real или proxy)
> ```
> Если в PC нет — действие зависит от способа загрузки:
> - `find`/`findById` → выполнить SELECT, положить **реальный** entity в PC.
> - `getReference` → создать **прокси**, положить в PC, SELECT отложить.
> - lazy-связь (`@ManyToOne(LAZY)`) → создать **прокси**, положить в PC.
> - JPQL/native `@Query` → выполнить SELECT, положить **реальный** entity в PC.
> Поэтому прокси на entity вы увидите только когда таргета нет в PC: новая сессия, после `clear`/`detach`, или его раньше в этой сессии никто не грузил.

>[!question]- Может ли объект, полученный напрямую через `repository.findById(...)`, оказаться прокси
> **Почти всегда нет, но строго говоря — да, в одном исключении.**
> Базовый случай (99% кода): `findById` делегируется в `entityManager.find(...)`, который либо отдаёт уже лежащий в PC managed-объект, либо делает SELECT и создаёт **реальный** entity. В обычном проекте в PC лежат реальные entity, поэтому:
> ```java
> EventBidFighter loadedBid = bidRepository.findById(bidId).get();
> loadedBid instanceof HibernateProxy;          // false
> loadedBid.getClass() == EventBidFighter.class; // true
> ```
> Исключение — **прокси этого же entity уже в PC** до вызова `findById`. Это бывает, если:
> 1. До этого вызвали `entityManager.getReference(EventBidFighter.class, bidId)` — там создаётся прокси без SELECT'а.
> 2. До этого лениво загрузили эту же запись как связь другого entity (`some.getBid()` с `@ManyToOne(LAZY)` или `@OneToOne(LAZY)`).
> 
> Тогда `find` по JPA-контракту вернёт **существующий managed-объект**, а это прокси:
> ```java
> EventBidFighter ref = entityManager.getReference(EventBidFighter.class, bidId);  // прокси в PC
> EventBidFighter loadedBid = bidRepository.findById(bidId).get();
> loadedBid instanceof HibernateProxy;  // true — это ТОТ ЖЕ прокси
> loadedBid == ref;                     // true
> ```
> На практике этот сценарий редок (нужно специально вызывать `getReference` или иметь lazy-связь, ведущую на ту же запись), поэтому правило большого пальца: **результат `findById` — реальный entity**.

>[!question]- А lazy-поля ВНУТРИ объекта из репозитория — они могут быть прокси
> **Да, это норма.** Сам `loadedBid` — реальный `EventBidFighter`, но его lazy-связи (`@ManyToOne(LAZY)`, `@OneToOne(LAZY)`) — это прокси, если их таргеты ещё не в PC:
> ```java
> EventBidFighter loadedBid = bidRepository.findById(bidId).get();  // реальный
> Event lazyEvent = loadedBid.getEvent();                            // прокси (если Event не в PC)
> Fighter lazyFighter = loadedBid.getFighter();                      // прокси
> ```
> Поэтому "объект из репозитория ≠ прокси" — корректное утверждение **только про сам корневой объект**. Его lazy-поля проксями быть вполне могут, и через них прокси проникают в код незаметно (через геттеры в сервисах, маппинг в DTO, сериализацию в JSON).

## Граничные случаи и подводные камни

>[!question]- Почему `proxy.getClass() != Event.class` даже если прокси — это "наследник Event"
> `getClass()` возвращает **реальный** класс объекта в JVM, а не "логический". Прокси — это **сгенерированный подкласс** `Event$HibernateProxy$xxx`, и `Class.equals` сравнивает по идентичности `Class<?>`-объекта. Поэтому:
> - `proxy instanceof Event` → `true` (по наследованию)
> - `proxy.getClass() == Event.class` → `false` (классы разные)
> Из-за этого IDE-generated `equals` с `if (getClass() != o.getClass()) return false` **всегда даёт false при сравнении прокси и обычного entity**. Канонический equals использует `Hibernate.getClass(o)` или проверку `instanceof HibernateProxy`.

>[!question]- Что вернёт `toString()` на неинициализированном Hibernate-прокси
> Зависит от того, как `toString` реализован:
> - Если стандартный `Object.toString` или Lombok без обращения к полям → что-то типа `Event@1a2b3c` (без SELECT'а).
> - Если `toString` (или Lombok `@ToString`) включает поле сущности → вызов геттера триггерит инициализацию прокси → выполнится SELECT.
> - Если сессия уже закрыта → `LazyInitializationException`.
> Поэтому в логах Hibernate-проектов важно: `@ToString.Exclude` на lazy-связях, иначе одно лог-сообщение может выполнить десятки запросов.

>[!question]- Может ли быть прокси на final-класс
> **Нет.** CGLIB/ByteBuddy создают прокси через наследование, а `final` класс наследовать нельзя. Поэтому Hibernate entity не должен быть `final` (но и от JPA-спеки требование то же). Spring AOP с `proxyTargetClass=true` на final-классе бросит ошибку при старте.
> Для прокси через интерфейсы (JDK) `final` на самом классе не мешает — главное, чтобы был интерфейс.

>[!question]- Что такое `Hibernate.unproxy(entity)` и когда его использовать
> Возвращает реальный entity, скрытый под прокси. Если передать обычный entity (не прокси), вернёт его же. Полезно в редких случаях, когда:
> - нужно сериализовать entity и не хочется тащить `HibernateLazyInitializer` в JSON (хотя обычно решают это `@JsonIgnore` или конвертацией в DTO),
> - нужно сравнить с `instanceof ConcreteEntityClass` (но лучше использовать `Hibernate.getClass(x)`).
> В большинстве случаев `unproxy` не нужен — пишите код так, чтобы он работал и с прокси, и с реальным entity (это и достигается каноническим `equals`/`hashCode`).

>[!question]- Почему прокси из Spring `@Transactional` не понимает аннотации на наследниках
> AOP-прокси создаётся для **конкретного класса бина**. Если у вас:
> ```java
> public class BaseService { @Transactional public void save(){} }
> public class UserService extends BaseService { @Override public void save(){} }
> ```
> Прокси оборачивает `UserService`. Что прокси перехватит и применит `@Transactional` к переопределённому методу — зависит от того, видит ли Spring аннотацию через наследование. Чтобы избежать сюрпризов, ставьте `@Transactional` **на конкретный класс или метод**, который вы хотите обернуть, а не только на родительский.

## Резюме

>[!question]- Что нужно запомнить о прокси одним абзацем
> Прокси-объект — это **сгенерированный в рантайме подкласс (или реализация интерфейса)** реального объекта, который перехватывает вызовы методов и добавляет/откладывает поведение. В Spring прокси — это механизм AOP: `@Transactional`, `@Async`, `@Cacheable` работают через автоматически создаваемые прокси (JDK Dynamic Proxy или CGLIB). В Hibernate прокси — это механизм lazy loading: `getReference()` и `@ManyToOne(LAZY)` возвращают `Entity$HibernateProxy$xxx`, который при первом обращении к полю выполнит SELECT. Прокси выглядит как target снаружи (`instanceof` проходит), но `getClass()` отличается — и это ломает наивные `equals`/`hashCode` ([[JPA equals hashCode]]). Из ограничений: класс/метод не должен быть `final`, нужен публичный/protected конструктор без аргументов, не работают вызовы `this.method()` через self-invocation, прокси нельзя сделать на `final`-класс.