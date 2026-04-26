# Domain Driven Design — Памятки для собеседования

---
1. [ ] что такое bounded context - чем определяются границы 
2. [ ] пример domain event в спринг 
3. [ ] что хорошо в правиле "бизнес логика находится в домене / сущности" а что плохо. 
## Основные концепции

>[!question]- Что такое DDD и зачем оно нужно?
> **Domain Driven Design** — подход к проектированию ПО, где **бизнес-логика (домен) является центром** всей архитектуры.
>
> **Зачем:**
> - Код отражает реальный бизнес, а не технические детали
> - Единый язык между разработчиками и бизнесом (Ubiquitous Language) - UI язык технички а не бвзинеса
> - Управление сложностью через разбиение на поддомены
>
> **Когда применять:** Сложные бизнес-домены с нетривиальными правилами. Не стоит использовать для CRUD-приложений.

>[!question]- где может находится репозитроий а где нет
> в application service не может (только в сервисе) ,а в сервисе может

>[!question]- INLINE правило
>Проблема inline-правила: читая auth.getLoginId().equals(eventPageDto.getOrganizerLogin()) — непонятно что это значит в бизнес-смысле. Читая event.isOrganizer(loginId) — понятно сразу. Имя метода = документация намерения.        

>[!question]- Что такое Ubiquitous Language (единый язык)?
> Общий язык, который **одинаково используют** разработчики, аналитики и бизнес.
>
> **Правила:**
> - Термины из бизнеса напрямую отражаются в коде (классы, методы, переменные)
> - Если бизнес говорит "Заказ" — в коде `Order`, а не `PurchaseRecord`
> - Язык живёт внутри **Bounded Context** — в разных контекстах один термин может означать разное
>
> **Пример:**
> - Бизнес: "Клиент оформляет Заказ, который проходит Проверку"
> - Код: `Customer.placeOrder()` → `Order.validate()`

>[!question]- Что такое Bounded Context?
> **Граница**, внутри которой модель домена и Ubiquitous Language имеют чёткое, однозначное значение.
> Например папка ``openevent``` - все что в ней находится отоносится только к отрытым событиям
>
> **Ключевое:**
> - Один термин в разных контекстах может означать разное (`User` в контексте Auth и в контексте Billing — разные объекты)
> - Каждый Bounded Context — отдельная модель, часто отдельный сервис/модуль
> - Контексты взаимодействуют через чётко определённые интерфейсы
>
> **Пример:** Интернет-магазин
> - Контекст (папка) **Catalog**: `Product` = название, описание, фото
> - Контекст (папка) **Warehouse**: `Product` = количество, ячейка хранения
> - Контекст (папка) **Billing**: `Product` = цена, НДС

>[!question]- Чем отличается Entity от Value Object?
> | | **Entity** | **Value Object** |
> |---|---|---|
> | Идентичность | По ID | По значению полей |
> | Изменяемость | Mutable | Immutable |
> | Сравнение | `id == id` | Все поля равны |
> | Пример | `User(id=1)` | `Money(100, "USD")` |
>
> **Entity:** объект с уникальной идентичностью, которая не меняется даже при изменении полей.
> ```
> User(id=1, name="Иван") == User(id=1, name="Иван Иванов") // true — тот же пользователь
> ```
>
> **Value Object:** описывает характеристику, не имеет ID, всегда immutable.
> ```
> Money(100, "USD") == Money(100, "USD") // true
> Money(100, "USD") != Money(200, "USD") // разные деньги
> ```
>
> **Признак Value Object:** если все поля одинаковые — это один и тот же объект.

>[!question]- Что такое Aggregate и Aggregate Root?
> **Aggregate** — кластер связанных объектов (Entity + Value Objects), которые рассматриваются как единица для изменений.
>
> **Aggregate Root** — единственная "точка входа" в агрегат. Только через него можно изменять объекты внутри.
>
> **Правила агрегатов:**
> 1. Внешний код обращается только к Root
> 2. Инварианты (бизнес-правила) поддерживаются на уровне всего агрегата
> 3. Агрегат сохраняется/загружается целиком
> 4. Ссылки между агрегатами — только через ID
>
> **Пример:**
> ```
> Order (Root)
>   ├── OrderItem (Entity)
>   ├── OrderItem (Entity)
>   └── ShippingAddress (Value Object)
> ```
> Нельзя изменить `OrderItem` напрямую — только через `Order.addItem()`, `Order.removeItem()`
> Например Event - **Aggregate Root**  , а 
> ```
> List<EventFiles> , List<EventDisciplines> ...
> ```
> это кластер связанных объектов (Entity + Value Objects) , внутри агрегатора не может быть других агрегаторов и у каждого агрегатра должен быть свой репозиторий, **Aggregate Root**  виден всей системе . 
> Агрегировать - значит соберать все агрегаты и использовать их в бизнес логике или переводить их в единый дто ответ на запрос

>[!question]- Что такое Domain Event?
> **Domain Event** — факт, который произошёл в домене и важен для бизнеса. Событие всегда в прошедшем времени.
>
> **Примеры:** `OrderPlaced`, `PaymentProcessed`, `UserRegistered`
>
> **Зачем:**
> - Слабая связанность между агрегатами и контекстами
> - Аудит-лог "что происходило"
> - Интеграция между Bounded Contexts
>
> **Как работает:**
> 1. Агрегат генерирует событие (`Order` → `OrderPlaced`)
> 2. Обработчики реагируют (`send email`, `update inventory`)
>
> **Отличие от команды:** Команда — намерение (`PlaceOrder`), событие — свершившийся факт (`OrderPlaced`).
> Примеры инфраструктуры 
> ```
> @EventListener, @TransactionalEventListener
> ```

>[!question]- Что такое Repository?
> **Repository** — абстракция для получения и сохранения агрегатов. Имитирует коллекцию объектов в памяти, скрывая детали хранения.
>
> **Правила:**
> - Репозиторий — для агрегатов, не для произвольных Entity
> - Интерфейс репозитория — в доменном слое, реализация — в инфраструктурном
> - Методы говорят на языке домена: `findByCustomerId()`, не `SELECT * FROM...`
>
> **Пример:**
> ```java
> // Доменный слой
> interface OrderRepository {
>     Order findById(OrderId id);
>     List<Order> findByCustomer(CustomerId customerId);
>     void save(Order order);
> }
>
> // Инфраструктурный слой
> class PostgresOrderRepository implements OrderRepository { ... }
> ```

>[!question]- Что такое Domain Service?
> **Domain Service** — операция из доменной логики, которая **не принадлежит ни одному конкретному агрегату**.
>
> **Признаки, что нужен Domain Service:**
> - Операция требует нескольких агрегатов
> - Операция не имеет естественного "владельца" среди Entity
>
> **Отличие от Application Service:**
> | | Domain Service | Application Service |
> |---|---|---|
> | Содержит | Бизнес-логику | Оркестрацию |
> | Зависит от | Домен | Домен + инфраструктура |
> | Пример | `TransferMoneyService` | `OrderApplicationService` |
>
> **Пример:**
> ```java
> // Domain Service — знает бизнес-правила перевода
> class MoneyTransferService {
>     void transfer(Account from, Account to, Money amount) {
>         from.debit(amount);
>         to.credit(amount);
>     }
> }
> ```

>[!question]- Что такое Application Service (Use Case)?
> **Application Service** — тонкий слой, который **оркестрирует** работу доменных объектов для выполнения конкретного use case.
>
> **Что делает:**
> - Загружает агрегаты через репозитории
> - Вызывает методы домена
> - Сохраняет результат
> - Публикует события
> - НЕ содержит бизнес-логику И МАППИНГ 
>
> **Пример:**
> ```java
> class PlaceOrderUseCase {
>     void execute(PlaceOrderCommand cmd) {
>         Customer customer = customerRepo.findById(cmd.customerId);
>         Order order = customer.placeOrder(cmd.items); // бизнес-логика в домене
>         orderRepo.save(order);
>         eventBus.publish(order.getDomainEvents());
>     }
> }
> ```

>[!question]- Что такое Factory в DDD?
> **Factory** — объект/метод, отвечающий за создание сложных агрегатов или Entity, инкапсулируя логику создания.
>
> **Когда нужна:**
> - Создание агрегата требует сложной логики
> - Нужно обеспечить инварианты при создании
>
> **Виды:**
> - Фабричный метод на самом агрегате (`Order.create(...)`)
> - Отдельный Factory класс (`OrderFactory`)
>
> **Пример:**
> ```java
> class Order {
>     static Order create(Customer customer, List<Item> items) {
>         if (items.isEmpty()) throw new DomainException("Заказ не может быть пустым");
>         Order order = new Order(OrderId.generate(), customer.getId());
>         items.forEach(order::addItem);
>         return order;
>     }
> }
> ```

>[!question]- Assembler это 
>(паттерн из книги Evans "Domain-Driven Design"). Его единственная ответственность — собрать DTO из нескольких доменных объектов. 
>```
>EventDescription + OpenEventPageModel + EventMedia → EventPageDto
>```
>- **Без состояния (Stateless):** Он не хранит данные, а просто пропускает их через себя.
>- - **Расположение:** Обычно находится на уровне **Application Layer** (Прикладной уровень), так как именно там происходит координация между внешним миром и доменом.
>- - **Разделение ответственности:** Позволяет менять структуру базы данных или домена, не ломая контракт API (вы просто меняете код внутри Assembler).
---

## Стратегическое проектирование

>[!question]- Что такое Context Map и какие паттерны взаимодействия контекстов бывают?
> **Context Map** — визуальная карта всех Bounded Context и отношений между ними.
>
> **Паттерны взаимодействия:**
>
> | Паттерн | Описание |
> |---|---|
> | **Shared Kernel** | Общая часть модели между двумя контекстами |
> | **Customer/Supplier** | Upstream поставляет, Downstream потребляет |
> | **Conformist** | Downstream копирует модель Upstream без изменений |
> | **ACL (Anti-Corruption Layer)** | Переводчик между моделями двух контекстов |
> | **Open Host Service** | Публичный API для множества потребителей |
> | **Published Language** | Общий формат обмена (JSON Schema, Protobuf) |
> | **Separate Ways** | Контексты вообще не связаны |
>
> **ACL** — самый важный: защищает доменную модель от "загрязнения" чужой моделью.

>[!question]- Что такое Core Domain, Supporting Domain, Generic Domain?
> Разбиение домена по **бизнес-ценности:**
>
> | Тип | Описание | Стратегия |
> |---|---|---|
> | **Core Domain** | Уникальное конкурентное преимущество бизнеса | Разрабатывать самим, лучшие люди |
> | **Supporting Domain** | Поддерживает Core, но не уникален | Разрабатывать самим или аутсорс |
> | **Generic Domain** | Стандартные задачи (email, auth, billing) | Купить готовое решение |
>
> **Пример (Uber):**
> - Core: алгоритм подбора водителя, динамическое ценообразование
> - Supporting: управление поездками, история
> - Generic: авторизация, платежи, SMS-уведомления

---

## Слои архитектуры

>[!question]- Опиши слои в DDD-архитектуре
> **Классические 4 слоя (от внешнего к внутреннему):**
>
> ```
> ┌─────────────────────────────┐
> │  Presentation / API         │  HTTP, gRPC, CLI
> ├─────────────────────────────┤
> │  Application                │  Use Cases, оркестрация
> ├─────────────────────────────┤
> │  Domain                     │  Entities, Aggregates, Services, Events
> ├─────────────────────────────┤
> │  Infrastructure             │  DB, External APIs, Messaging
> └─────────────────────────────┘
> ```
>
> **Ключевое правило:** зависимости направлены **внутрь**. Domain ни от чего не зависит. Infrastructure зависит от Domain (через интерфейсы).
>
> В **Hexagonal Architecture (Ports & Adapters)** та же идея: домен в центре, снаружи — адаптеры (REST, DB, Queue).

>[!question]- В чём разница между Layered Architecture и Hexagonal Architecture в контексте DDD?
> **Layered Architecture:**
> - Строгая иерархия слоёв сверху вниз
> - Presentation → Application → Domain → Infrastructure
> - Минус: Infrastructure в самом низу, но Domain зависит от его интерфейсов
>
> **Hexagonal (Ports & Adapters):**
> - Домен в центре, адаптеры снаружи
> - **Ports** — интерфейсы домена (Repository, EventBus)
> - **Adapters** — реализации (PostgresRepo, KafkaEventBus)
> - Явно выражает: домен ни от чего не зависит
>
> Оба подхода совместимы с DDD. Hexagonal лучше выражает независимость домена.

---

## Задачки

>[!question]- Задача: Определи, что является Entity, а что Value Object
> **Дано:** Система бронирования отелей. Есть: `Бронирование`, `Номер`, `Дата заезда`, `Период проживания (с/по)`, `Деньги (сумма + валюта)`, `Гость`, `Адрес гостиницы`.
>
> **Ответ:**
>
> | Объект | Тип | Почему |
> |---|---|---|
> | Бронирование | Entity | Уникальный ID, меняется статус |
> | Номер | Entity | Конкретный физический номер с ID |
> | Гость | Entity | Конкретный человек с историей |
> | Дата заезда | Value Object | Просто значение, нет ID |
> | Период (с/по) | Value Object | Описывает характеристику, immutable |
> | Деньги (100 USD) | Value Object | Нет идентичности, сравнивается по значению |
> | Адрес гостиницы | Value Object | Описание места, нет ID |

>[!question]- Задача: Найди нарушения правил агрегатов
> **Дано:**
> ```java
> class Order {
>     List<OrderItem> items;
>     Customer customer; // прямая ссылка на другой агрегат
> }
>
> class OrderItem {
>     Product product; // прямая ссылка на другой агрегат
> }
>
> // Где-то в сервисе:
> order.getItems().get(0).setPrice(newPrice); // прямой доступ к вложенному Entity
> ```
>
> **Нарушения:**
> 1. `Order` хранит прямую ссылку на `Customer` (другой агрегат) → нужно хранить `CustomerId`
> 2. `OrderItem` хранит прямую ссылку на `Product` → нужно `ProductId` + snapshot цены
> 3. Изменение `OrderItem` напрямую, минуя `Order` → нарушение инкапсуляции агрегата
>
> **Исправление:**
> ```java
> class Order {
>     CustomerId customerId; // ссылка по ID
>     List<OrderItem> items; // внутренние Entity — ок
>
>     void changeItemPrice(OrderItemId itemId, Money newPrice) {
>         // только через метод Root
>         items.find(itemId).updatePrice(newPrice);
>         // возможно проверить бизнес-правила всего Order
>     }
> }
> ```

>[!question]- Задача: Спроектируй Bounded Contexts для e-commerce платформы
> **Задача:** Выдели Bounded Contexts для интернет-магазина с: каталогом товаров, заказами, оплатой, доставкой, отзывами, уведомлениями.
>
> **Вариант решения:**
>
> ```
> [Catalog Context]        — Product(id, name, description, images, categories)
> [Inventory Context]      — Product(id, stock_qty, warehouse_location)
> [Order Context]          — Order, OrderItem(productId, price snapshot), Customer
> [Payment Context]        — Payment, Transaction, Invoice
> [Shipping Context]       — Shipment, DeliveryAddress, Carrier
> [Reviews Context]        — Review, Rating, ProductId
> [Notification Context]   — Template, Channel, Recipient
> ```
>
> **Взаимодействие:**
> - `Order` → `Catalog`: ACL (берёт snapshot цены при оформлении)
> - `Order` → `Payment`: Customer/Supplier (Order инициирует, Payment обрабатывает)
> - `Order` → `Shipping`: Domain Event `OrderPaid` → создаёт Shipment
> - `Order` → `Notification`: Domain Events (OrderPlaced, OrderShipped)

>[!question]- Задача: Что нарушено в этом коде, как исправить?
> ```java
> class OrderService {
>     void placeOrder(Long customerId, List<Long> productIds) {
>         // Бизнес-правило: VIP клиент получает скидку 10%
>         Customer customer = customerRepo.findById(customerId);
>         List<Product> products = productRepo.findAllById(productIds);
>
>         double total = products.stream()
>             .mapToDouble(p -> p.getPrice())
>             .sum();
>
>         if (customer.isVip()) {
>             total = total * 0.9; // бизнес-логика в сервисе!
>         }
>
>         Order order = new Order();
>         order.setCustomerId(customerId);
>         order.setTotal(total);
>         order.setStatus("PENDING");
>         orderRepo.save(order);
>     }
> }
> ```
>
> **Проблемы:**
> 1. Бизнес-логика (скидка VIP) — в Application Service, а не в домене
> 2. `new Order()` + `setters` — нет инвариантов при создании, нет Factory
> 3. `"PENDING"` — строка вместо enum/Value Object
> 4. `total` — примитив, а не `Money` Value Object
>
> **Исправление:**
> ```java
> // Логика скидки — в Order или в Domain Service
> class Order {
>     static Order place(Customer customer, List<OrderItem> items) {
>         Money subtotal = items.stream()
>             .map(OrderItem::price)
>             .reduce(Money.ZERO, Money::add);
>         Money total = customer.applyDiscount(subtotal);
>         return new Order(OrderId.generate(), customer.getId(),
>                          items, total, OrderStatus.PENDING);
>     }
> }
> ```

>[!question]- Задача: Объясни, как Domain Events помогают избежать связанности
> **Ситуация:** При оформлении заказа нужно: списать деньги, уменьшить остаток на складе, отправить email, начислить бонусы.
>
> **Плохой подход (сильная связанность):**
> ```java
> class PlaceOrderUseCase {
>     void execute(cmd) {
>         Order order = ...; // создать заказ
>         paymentService.charge(order);      // прямая зависимость
>         inventoryService.reserve(order);   // прямая зависимость
>         emailService.sendConfirmation();   // прямая зависимость
>         loyaltyService.addPoints();        // прямая зависимость
>     }
> }
> ```
>
> **Хороший подход (через Domain Events):**
> ```java
> class PlaceOrderUseCase {
>     void execute(cmd) {
>         Order order = Order.place(...);
>         orderRepo.save(order);
>         eventBus.publish(new OrderPlaced(order.getId(), ...));
>         // всё, больше ничего не знаем
>     }
> }
>
> // Каждый обработчик независим:
> @EventHandler OrderPlaced → PaymentHandler.charge()
> @EventHandler OrderPlaced → InventoryHandler.reserve()
> @EventHandler OrderPlaced → EmailHandler.sendConfirmation()
> @EventHandler OrderPlaced → LoyaltyHandler.addPoints()
> ```
>
> **Выгоды:** Добавить новый обработчик = не трогать PlaceOrderUseCase. Каждый контекст сам решает, как реагировать.
