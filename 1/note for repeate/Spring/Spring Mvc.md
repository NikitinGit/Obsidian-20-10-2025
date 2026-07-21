1. [x] правильно ли сказать что томкат может работать как на http так и на websocket только при handshake  запрос отправляется по http а потом меняется на websocket и переподключения не происходит а просиходит просто upgrade? - да
2. [x] А если между клиентом и томкатом стоит nginx который после handshake апгрейдит протокол с http на websocket , то что после этого происходит - на томкат прилетает запрос handshake? Если прилетает на каком протоколе? - на том же http  прилетает запрос handshake 
3. [x] томкат - это веб сервер (потому что принято под сервером понимать компьютер, способный принимать запросы), который спосообен работать только на тех протоколах, которые построены по верх TCP (не надежные протоколы сетевого уровня типа UDP / rudp он не способен обрабатывать) ? И тоже касается любого другого котейнера сервелетов ? - да
4. [x] если томкат - контейнер сервлетов - но по сути у него 1 сервлет DispatcherServlet , то почему во множественном числе "контейнер сервлетов" - там могут быт еще другие сервлеты ? да могут, и так исторически  и было 
5. [x] а чем пул потоков подключения к БД в  hikarypool отличается от того пула потоков который мы обсуждали?
6. [x] если томкат - это контейнер сервлетов, то почему он фильтрует запросы Spring Security  ? Ведь без томкат сервера приложение вообще ни какие запросы не принимает ? Чем тогда являются фильтры в томкат если не сервлетами (можно же подумать что фильтр это тоже обработка запроса - получени яв хедере кук и по ним определять пускать дальше на обработку запрос или нет) ?
7. [x] как настраивать пул потоков в application.properties - как определить цифры 
8. [x] то есть чтобы определить какие настройки надо написать application.properties надо определить на какой машине будет работать спринг приложение, какая архитектура приложения (какая БД, какие протоколы используется по мимо http (websocket), какие требования по обработке запроса (за какое время он должен исполняться), промониторить логи (елк стек, графана) ?
9. [x] у каждого микросервиса на Spring должен обслуживаться своим встроенным в спринг сервлет контейнером (томкат например) или они могут обслуживаться оддним токатом? Как это принято делать ? - принято для каджого свой томкат
10. [ ] "Он отвязывает обработку запроса от конкретного потока: поток, принявший запрос, сразу возвращается в пул, а запись в открытый response происходит позже, из другого места, когда появляются новые данные." - где он хранится тогда ?
11. [ ] Важное замечание о Виртуальных потоках (Java 21+) и прочти статью https://medium.com/@gaddamnaveen192/we-replaced-500-tomcat-threads-with-virtual-threads-the-results-shocked-us-12cd9afe4a15 
12. [ ] **Асинхронные методы (`@Async`)**:
13. [ ] application.properties - https://habr.com/ru/articles/740802/ 
14. [ ] Spring webflux 
# Spring MVC

>[!question]- Что такое Spring MVC?
> Spring MVC — модуль `spring-webmvc`, реализующий паттерн **Model-View-Controller** для HTTP-приложений. Подтягивается через `spring-boot-starter-web` (включает встроенный Tomcat, Jackson, валидацию).
> Центральный класс — **`DispatcherServlet`**: единая точка входа для HTTP-запросов, распределяет их по контроллерам, использует `HandlerMapping` (по аннотациям), `HandlerAdapter`, `ViewResolver` и т.д.

>[!question]- что входит в Tomcat
>Servlet — конечные обработчики (DispatcherServlet, H2-консоль и т.д.);
> Filter — цепочка "до/после" вокруг вызова сервлета (Spring Security и др.);
> плюс ещё ServletContextListener/другие *Listener-интерфейсы — колбэки на события уровня всего приложения (например, "контекст приложения поднялся" / "контекст остановлен") — тоже управляются контейнером, просто мы их пока не разбирали.  


>[!question]- `@Controller` vs `@RestController`
> - **`@Controller`** — возвращает имя view (Thymeleaf, JSP). Используется в классических MVC-приложениях с серверным рендерингом.
> - **`@RestController`** — `@Controller + @ResponseBody`. Возвращает тело ответа напрямую (JSON через Jackson). Используется для REST-API.

## URL-маппинг

>[!question]- `@RequestMapping`, `@GetMapping`, `@PostMapping` и т.д.
> - **`@RequestMapping`** — универсальная, поддерживает все методы и атрибуты. Можно вешать на класс (общий префикс) и на метод.
> - **`@GetMapping`**, **`@PostMapping`**, **`@PutMapping`**, **`@DeleteMapping`**, **`@PatchMapping`** — шорткаты для `@RequestMapping(method = ...)`.
>
> Пример комбинации:
> ```java
> @RestController
> @RequestMapping("/api/test")           // общий префикс для всех методов
> public class AspectController {
>     @GetMapping("/1")                  // итоговый путь /api/test/1
>     public ResponseEntity<String> endPointN1() { ... }
> }
> ```

>[!question]- Что такое URL placeholder и как читать `/3/{eventId}/4/{userId}`?
> **Placeholder** в URL-паттерне — это **фигурные скобки `{...}`**, обозначающие "сюда подставится значение из реального URL".
>
> ```java
> @GetMapping("/3/{eventId}/4/{userId}")
> //             ^^^^^^^^^   ^^^^^^^^
> //             placeholder placeholder
> ```
> При запросе `GET /api/test/3/100/4/200`:
> - `{eventId}` → `100`
> - `{userId}` → `200`
>
> Эти значения достаются через `@PathVariable Integer eventId, @PathVariable Integer userId`.
>
> **Аналогия:** как `%s` в `String.format` или `{}` в SLF4J — "шаблон с дыркой, в которую вставляется значение".

## `@PathVariable` vs `@RequestParam`

>[!question]- В чём разница?
> | | `@PathVariable` | `@RequestParam` |
> |---|---|---|
> | Где параметр в URL | в **пути** (между слешами) | в **query string** (после `?`) |
> | Обозначение в `@GetMapping` | `{name}` placeholder | без обозначения |
> | Пример URL | `/api/users/100/orders/5` | `/api/users?id=100&limit=10` |
> | Аннотация | `@PathVariable Integer userId` | `@RequestParam Integer userId` |
> | Семантика | идентификация ресурса | фильтры, опции, флаги |
>
> **Пример обоих стилей:**
> ```java
> // PathVariable
> @GetMapping("/users/{userId}/orders/{orderId}")
> public ResponseEntity<?> getOrder(@PathVariable Long userId, @PathVariable Long orderId)
>
> // RequestParam
> @GetMapping("/search")
> public ResponseEntity<?> search(@RequestParam String query, @RequestParam(defaultValue = "10") int limit)
> ```

>[!question]- Распространённая ошибка — смешать `{...}` и `@RequestParam`
> ```java
> @GetMapping("/3/{eventId}/4/{userId}")              // путь с placeholder'ами
> public ResponseEntity<?> handle(
>         @RequestParam Integer eventId,              // но берём из query string!
>         @RequestParam Integer userId) { ... }
> ```
> Что произойдёт:
> 1. URL должен матчиться по форме `/3/{любое}/4/{любое}` — иначе **404**.
> 2. Эти path-значения **никуда не привяжутся** (нет `@PathVariable`).
> 3. `@RequestParam eventId` пойдёт искать `?eventId=...`. Не нашёл — **400 Bad Request**.
>
> Чтобы попасть в такой эндпоинт пришлось бы передать всё дважды: `/api/test/3/foo/4/bar?eventId=100&userId=200`. Путь `foo`/`bar` обязателен (иначе 404), но не используется. Это запутывающе и почти всегда баг.
>
> **Правило:** либо `{...}` + `@PathVariable`, либо без `{...}` + `@RequestParam`. Не смешивай.

## `@RequestParam` детально

>[!question]- Что задаёт значение в `@RequestParam("manId")`?
> Атрибут `value` (он же `name`) — **имя HTTP-параметра в URL**, не имя переменной.
>
> | Объявление | Что ищет в URL | Локальная переменная |
> |---|---|---|
> | `@RequestParam Integer userId` | `?userId=...` | `userId` |
> | `@RequestParam("manId") Integer userId` | `?manId=...` | `userId` |
>
> То есть параметр аннотации — это **внешнее имя в API**, а не "псевдоним переменной". Снаружи параметр называется `manId`, внутри — `userId`.

>[!question]- Когда нужен явный алиас в `@RequestParam`?
> **1. Snake_case в API, camelCase в Java:**
> ```java
> @RequestParam("user_id") Integer userId
> @RequestParam("event_date") LocalDate eventDate
> ```
> **2. Зарезервированные слова Java:**
> ```java
> @RequestParam("class") String clazz
> ```
> **3. Старый контракт API, который нельзя ломать:**
> ```java
> @RequestParam("manId") Integer userId   // снаружи manId (legacy), внутри userId (чистый код)
> ```
> **4. Без флага `-parameters`** имена параметров стираются в `arg0, arg1`. Тогда явное имя обязательно. В Spring Boot `-parameters` включён по умолчанию, но если кастомный компиляторный конфиг — может потребоваться.
>
> **5. Защита от случайного переименования** — если кто-то отрефакторит `userId → user`, без алиаса API сломается молча. С алиасом — переменная переименуется, контракт API останется.

>[!question]- Полный список атрибутов `@RequestParam`
> | Атрибут | Назначение | Пример |
> |---|---|---|
> | `value` / `name` | Имя HTTP-параметра | `@RequestParam("user_id") Integer userId` |
> | `required` | Обязателен ли параметр (по умолчанию `true`) | `@RequestParam(required = false) Integer page` → если нет, будет `null` |
> | `defaultValue` | Значение по умолчанию, если параметра нет | `@RequestParam(defaultValue = "10") Integer limit` |
>
> Полная форма:
> ```java
> @RequestParam(value = "user_id", required = false, defaultValue = "0") Integer userId
> ```
> При `defaultValue` параметр автоматически становится `required = false`.

>[!question]- `@RequestParam` и Map
> Можно собрать **все** query-параметры в `Map`:
> ```java
> @GetMapping("/search")
> public String search(@RequestParam Map<String, String> allParams) { ... }
> ```
> Аналогично для form-параметров через `@RequestParam MultiValueMap<String, String>`.

## `@PathVariable` детально

>[!question]- Атрибуты `@PathVariable`
> | Атрибут | Назначение |
> |---|---|
> | `value` / `name` | Имя placeholder'а (если отличается от переменной) |
> | `required` | Обязателен ли (только при `Optional<>` или составных шаблонах) |
>
> ```java
> @GetMapping("/users/{user_id}")
> public ResponseEntity<?> get(@PathVariable("user_id") Long userId)
> ```

>[!question]- Regex в path-placeholder'ах
> Можно ограничить значение регуляркой прямо в шаблоне:
> ```java
> @GetMapping("/users/{userId:\\d+}")                 // только цифры
> @GetMapping("/files/{filename:.+}")                  // включая точки (без regex точка не захватывается полностью)
> ```

## `Ambiguous mapping` — частая ошибка

>[!question]- Ошибка `Ambiguous mapping. Cannot map ... to {GET [/...]}`
> **Полный текст:**
> ```
> org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'requestMappingHandlerMapping'
>   ... Ambiguous mapping. Cannot map 'aspectController' method
>   ...#endPointN3()
>   to {GET [/api/test/3]}: There is already 'aspectController' bean method
>   ...#endPointParamN1(Integer, Integer) mapped.
> ```
>
> **Причина:** два метода контроллера на **одинаковый URL + HTTP-метод**. Spring не может выбрать → контекст не поднимается, приложение падает на старте.
>
> **Важно:** `@RequestParam` **НЕ участвует в URL-маппинге**. Query string — не часть пути. `/3?eventId=1` и `/3` для Spring это **один и тот же маппинг**.
>
> **Решение:** дать одному из методов другой путь.
>
> **Альтернатива** — различать по query-параметрам через атрибут `params`:
> ```java
> @GetMapping(value = "/3", params = "eventId")     // срабатывает, если ?eventId=... есть
> @GetMapping(value = "/3", params = "!eventId")    // срабатывает, если ?eventId=... НЕТ
> @GetMapping(value = "/3", params = "type=admin")  // срабатывает только при ?type=admin
> ```
> Аналогичные атрибуты: `headers`, `consumes` (Content-Type), `produces` (Accept).

## Стандартные параметры обработчика

>[!question]- Что может стоять в параметрах метода контроллера
> Spring MVC резолвит параметры через **`HandlerMethodArgumentResolver`**. Самые частые:
>
> | Тип/аннотация | Что приходит |
> |---|---|
> | `@PathVariable` | значение из placeholder'а пути |
> | `@RequestParam` | значение из query string или form-data |
> | `@RequestBody` | десериализованное тело запроса (JSON → объект через Jackson) |
> | `@RequestHeader` | значение HTTP-заголовка |
> | `@CookieValue` | значение cookie |
> | `@ModelAttribute` | объект, собранный из form-полей |
> | `HttpServletRequest` / `HttpServletResponse` | низкоуровневый доступ |
> | `Model` / `ModelMap` | передача данных во view (для классического `@Controller`) |
> | `Principal` / `Authentication` | текущий пользователь (Spring Security) |
> | `BindingResult` | результат валидации, должен идти **сразу** после валидируемого объекта |

## Возврат данных и `ResponseEntity`

>[!question]- Что возвращать из контроллера?
> 1. **Объект** (`User`, `List<Order>`) → Jackson сериализует в JSON, статус 200.
> 2. **`ResponseEntity<T>`** — полный контроль над статусом, заголовками, телом:
>    ```java
>    return ResponseEntity.status(HttpStatus.CREATED)
>        .header("X-Custom", "value")
>        .body(new User(...));
>    ```
>    Шорткаты: `.ok(body)`, `.notFound().build()`, `.badRequest().body(...)`.
> 3. **`void`** + `@ResponseStatus(HttpStatus.NO_CONTENT)` — для эндпоинтов без тела.
> 4. **`String`** в `@Controller` — имя view (Thymeleaf). В `@RestController` — просто строка как тело.

>[!question]- Глобальная обработка исключений — `@ControllerAdvice`
> ```java
> @RestControllerAdvice
> public class GlobalExceptionHandler {
>     @ExceptionHandler(EntityNotFoundException.class)
>     public ResponseEntity<ErrorResponse> handle(EntityNotFoundException e) {
>         return ResponseEntity.status(HttpStatus.NOT_FOUND)
>             .body(new ErrorResponse(e.getMessage()));
>     }
> }
> ```
> Применяется ко всем контроллерам (можно ограничить через `basePackages`, `assignableTypes`, аннотации).

## Валидация

>[!question]- `@Valid` + `BindingResult`
> ```java
> @PostMapping("/users")
> public ResponseEntity<?> create(@Valid @RequestBody UserDto dto, BindingResult result) {
>     if (result.hasErrors()) {
>         return ResponseEntity.badRequest().body(result.getAllErrors());
>     }
>     ...
> }
> ```
> На полях DTO — `@NotNull`, `@NotBlank`, `@Email`, `@Min`, `@Max`, `@Size`, `@Pattern` и т.д. (`jakarta.validation.constraints`).
> Без `BindingResult` Spring сам бросит `MethodArgumentNotValidException` (его удобно ловить через `@ExceptionHandler`).

## Полезное

>[!question]- Как Spring MVC обрабатывает запрос (упрощённо)
> 1. **`DispatcherServlet`** ловит запрос.
> 2. **`HandlerMapping`** (обычно `RequestMappingHandlerMapping`) находит метод контроллера по URL + HTTP-методу + аннотациям.
> 3. **`HandlerAdapter`** вызывает метод, резолвит параметры через `HandlerMethodArgumentResolver`.
> 4. **Возвращаемое значение** обрабатывается `HandlerMethodReturnValueHandler`:
>    - `@ResponseBody` → сериализация через `HttpMessageConverter` (Jackson для JSON).
>    - имя view → `ViewResolver` → рендеринг.
> 5. **Фильтры/Interceptors** могут вмешиваться до/после.
> 6. **`@ControllerAdvice`** ловит исключения, не обработанные внутри.

>[!question]- Какой URL соответствует методам с разными комбинациями
> ```java
> @RestController
> @RequestMapping("/api/test")
> public class AspectController {
>
>     @GetMapping("/1/{eventId}/2/{userId}")
>     handle1(@PathVariable Integer eventId, @PathVariable Integer userId)
>     // GET /api/test/1/100/2/200
>
>     @GetMapping("/param")
>     handle2(@RequestParam Integer eventId, @RequestParam Integer userId)
>     // GET /api/test/param?eventId=100&userId=200
>
>     @GetMapping("/mixed/{eventId}")
>     handle3(@PathVariable Integer eventId, @RequestParam Integer userId)
>     // GET /api/test/mixed/100?userId=200
>
>     @PostMapping("/body")
>     handle4(@RequestBody UserDto dto)
>     // POST /api/test/body  + JSON в теле
> }
> ```

>[!question]- Spring hateoas
>библиотека, которая позволяет добавлять в  рест-ответы ссылки на возможные действия 
>```{
  "id": 1,
  "name": "Alex",
  "_links": {
>"self": { "href": "http://localhost:8080/users/1" },
>"all": { "href": "http://localhost:8080/users" }
  }
}
>```
>подходит когда 
>**публичных API**, где важно самоописание (например, GitHub API, PayPal API);
>когда клиентам не известен весь список урл адресов
>бизнес логика часто менятея и API должен подсказывать клиентучто делать 

>[!question]- **Spring WebFlux**
>потоки не блокируются и могут переиспользоваться при запросах
>больше подходит для стриминга

# Пул потоков

>[!question]- Как Spring абстрагирует пулы потоков — `TaskExecutor` / `ThreadPoolTaskExecutor`
> Управление пулами потоков в Spring абстрагировано через интерфейс **`TaskExecutor`** (аналог стандартного Java `Executor`). Главная и самая часто используемая реализация — **`ThreadPoolTaskExecutor`**, которая оборачивает обычный `java.util.concurrent.ThreadPoolExecutor` и позволяет настраивать его декларативно через IoC-контейнер (бином или через `application.properties`), а не императивным кодом.

>[!question]- Три ключевых параметра `ThreadPoolTaskExecutor`
> - **`CorePoolSize`** — стартовое и минимальное число постоянно живущих потоков в пуле.
> - **`QueueCapacity`** — ёмкость очереди задач (запросов). Когда все core-потоки заняты, новые задачи встают сюда.
> - **`MaxPoolSize`** — максимально допустимое число потоков. Пул расширяется до этого значения **только когда очередь уже полностью заполнена**, а новые задачи продолжают поступать.
>
> Порядок роста: сначала занимаются core-потоки → потом заполняется очередь → и только когда очередь переполнена, создаются дополнительные потоки сверх core (вплоть до max). Если и max занят, и очередь полна — задача отклоняется (`RejectedExecutionException` по умолчанию).

>[!question]- Настройка пула как Spring-бин
> ```java
> @Configuration
> public class AsyncConfig {
>
>     @Bean(name = "customTaskExecutor")
>     public Executor taskExecutor() {
>         ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
>         executor.setCorePoolSize(5);
>         executor.setMaxPoolSize(10);
>         executor.setQueueCapacity(25);
>         executor.setThreadNamePrefix("MyAsyncThread-");
>         executor.initialize();
>         return executor;
>     }
> }
> ```

>[!question]- Настройка через `application.properties` (Spring Boot)
> ```properties
> # Для асинхронных задач (@Async)
> spring.task.execution.pool.core-size=5
> spring.task.execution.pool.max-size=10
> spring.task.execution.pool.queue-capacity=25
> spring.task.execution.thread-name-prefix=async-task-
>
> # Для планировщика задач (@Scheduled)
> spring.task.scheduling.pool.size=2
> spring.task.scheduling.thread-name-prefix=scheduling-
> ```

>[!question]- Где в Spring реально используются пулы потоков
> - **`@Async`** — вынос тяжёлых/долгих операций (отправка писем, генерация отчётов) в фоновые потоки. Требует `@EnableAsync` на конфигурации.
> - **`@Scheduled`** — регулярное выполнение задач по расписанию/cron. Требует `@EnableScheduling`.
> - **Обработка HTTP-запросов** — этим управляет не сам Spring, а встроенный веб-сервер (Tomcat по умолчанию в Spring Boot). У Tomcat свой пул потоков (по умолчанию до ~200 воркеров, `server.tomcat.threads.max`), и именно поэтому **`@Service`/`@Controller`-бины должны быть stateless**: это singleton-экземпляры, которые параллельно вызываются разными потоками из пула Tomcat на разных запросах — изменяемое **поле** такого бина (не путать с локальной переменной метода — она у каждого потока своя на стеке) станет общим ресурсом и источником гонки данных.

>[!question]- Как настраивать `server.tomcat.*` в `application.properties` — как определить цифры, а не просто скопировать дефолты
> ```properties
> server.tomcat.threads.max=200        # потолок рабочих потоков (дефолт 200)
> server.tomcat.threads.min-spare=10   # минимум "тёплых" потоков, живущих всегда (дефолт 10)
> server.tomcat.accept-count=100       # очередь TCP-соединений, ждущих свободный поток (дефолт 100)
> server.tomcat.max-connections=8192   # потолок вообще принятых соединений (дефолт 8192)
> ```
> Поправка к калауту про `ThreadPoolTaskExecutor` выше: у Tomcat `accept-count` — это **не JDK-очередь задач**, а `backlog` на уровне самого TCP `accept()` (тот же параметр, что мы разбирали в `AsyncTcpServer`/[[TCP]]). Когда все `max` потоков заняты, новые TCP-соединения не отклоняются сразу, а копятся в этой очереди на уровне ОС/сокета, ожидая освобождения потока.
>
> **Как реально определить цифры (не гадание):**
> 1. **Формула для I/O-bound нагрузки** (HTTP-обработка почти всегда такая — ждём БД, внешние API, диск): `потоков ≈ ядра CPU × (1 + время_ожидания / время_вычислений)`. Пока поток ждёт ответ от БД, CPU простаивает зря — больше потоков, чем ядер, позволяет другим потокам использовать CPU в этот момент.
> 2. **Соотношение ожидание/вычисление измеряется, а не угадывается** — нужен реальный профайлинг (Actuator + Micrometer + Prometheus/Grafana или лог таймингов): сколько времени запрос реально ждёт БД/внешний сервис против чистого CPU-времени.
> 3. **Узкое место ниже по цепочке — жёсткий потолок сверху.** Прямая связь с [[HikariCP]]: если `threads.max` сильно больше `maximumPoolSize` DB-пула, лишние потоки просто блокируются в очереди за соединением к БД — больше реальной пропускной способности это не даёт, только больше потоков, стоящих в очереди друг за другом.
> 4. **Дефолт 200 — не рассчитанное под ваше приложение число**, а разумный средний дефолт, безопасно работающий для большинства небольших/средних приложений без всякой настройки.
> 5. **Практический метод — нагрузочное тестирование, а не формула наперёд.** Дефолт → `JMeter`/`Gatling`/`k6` под ожидаемой нагрузкой → метрики (активные потоки, растёт ли очередь `accept-count`, латентность, CPU) → подкрутка → повтор. Растущая очередь `accept-count` при устойчивой (не всплесковой) нагрузке — сигнал нехватки потоков/ресурсов ниже по цепочке, а не повод просто увеличить саму очередь (это маскирует проблему задержкой, а не решает её).
>
> **Виртуальные потоки (Java 21+, `spring.threads.virtual.enabled=true`)** меняют всю картину — модель "заранее посчитанного потолка `max`" теряет смысл, т.к. виртуальные потоки достаточно дёшевы, чтобы не упираться в фиксированный потолок платформенных потоков для I/O-bound нагрузки.

>[!question]- `spring.task.scheduling.pool.size` vs `@Async` — обе про "долгая задача блокирует другие", но разные пулы
> Обе крутятся вокруг пула потоков, но управляют **разными** пулами и решают проблему на разных уровнях.
>
> | | `spring.task.scheduling.pool.size` | `@Async` |
> |---|---|---|
> | Какой пул трогает | `TaskScheduler` — пул, который **вызывает** `@Scheduled`-методы по расписанию | `taskExecutor` (`ThreadPoolTaskExecutor` из `@Async`-конфига) — отдельный пул, который **выполняет** тело метода |
> | Что решает | Сколько триггеров `@Scheduled` могут сработать одновременно | Освобождает вызывающий поток сразу после вызова, не дожидаясь завершения метода |
> | Дефолт без настройки | **1 поток на ВСЕ `@Scheduled`-методы проекта** (если `TaskScheduler`-бин не задан и `pool.size` не указан) — самая частая засада | Требует `@EnableAsync` + свой Executor-бин; не привязан именно к `@Scheduled` |
>
> **Конкретный пример из проекта strikerstat:** `ChallengeNotificationWorker#dispatchExternalNotifications` и `PaymentsScheduling#runEveryFiveSeconds` сидели на одном общем scheduler-потоке (дефолтный pool size = 1) — если зависал внешний SMS/email-провайдер внутри воркера уведомлений, блокировались вообще все `@Scheduled`-задачи проекта, включая проверку платежей. Fix: `spring.task.scheduling.pool.size=8` в `application.properties` — теперь у планировщика 8 потоков, задачи не ждут друг друга в очереди.
>
> **Альтернатива (уже используется в этом же проекте) — `VideoConversionService`:**
> ```java
> @Async
> @Scheduled(fixedDelay = 60000, initialDelay = 30000)
> @Transactional
> public void triggerVideoConversionJob() { ... }
> ```
> Здесь `@Scheduled` триггерит вызов как обычно (через прокси), но благодаря `@Async` сам вызов сразу улетает в `taskExecutor`, а scheduler-поток возвращается в свой пул почти мгновенно — независимо от того, сколько реально длится метод. Работает даже при `pool.size=1`: scheduler-поток вообще не блокируется, потому что реальная работа идёт на другом (executor-) пуле.
>
> **Нюансы `@Async`, которых нет у простого увеличения `pool.size`:**
> 1. Метод должен возвращать `void`/`Future`/`CompletableFuture` — не любой тип.
> 2. Работает только через **внешний** вызов (прокси) — self-invocation внутри того же класса `@Async` проигнорирует. `@Scheduled` вызывает извне через прокси планировщика, так что комбинация рабочая.
> 3. Метод теперь делит `taskExecutor` с остальным `@Async`-кодом в приложении — это отдельный, общий пул, не выделенный именно под эту задачу (в отличие от `pool.size`, выделенного только под сам scheduler).
> 4. Раз scheduler-поток освобождается сразу, а не после завершения работы — `fixedDelay`/`fixedRate` перестают гарантировать **неперекрывающиеся запуски**: следующий тик может стартовать, пока предыдущий асинхронный вызов ещё не закончился. Нужен свой guard в теле метода, если это важно (в `VideoConversionService` для этого есть проверка числа уже запущенных ffmpeg-процессов).
> 5. Необработанное исключение в `void`-`@Async`-методе не долетает до вызывающего кода — уходит в `AsyncUncaughtExceptionHandler` (по умолчанию просто логируется), а не пробрасывается как при обычном вызове.
>
> **Вывод:** `pool.size` — грубее, но проще и без побочных эффектов, лечит "все `@Scheduled` делят один поток" целиком и сразу. `@Async` — точечнее (освобождает scheduler-поток для КОНКРЕТНОЙ долгой/сетевой задачи, не трогая остальные), но добавляет риск перекрывающихся запусков и делит с остальным кодом общий executor-пул. Их можно комбинировать: держать `pool.size` с запасом, и ДОПОЛНИТЕЛЬНО навесить `@Async` на самую рискованную (сетевую/зависающую) задачу — кандидат в этом проекте: `ChallengeNotificationWorker`, дёргающий внешние SMS/email API.

>[!question]- Виртуальные потоки (Java 21+, Project Loom)
> На Java 21+ вместо классического ограниченного пула можно включить виртуальные потоки — они очень лёгкие, создаются динамически под каждую задачу и не требуют лимита пулом:
> ```properties
> spring.threads.virtual.enabled=true
> ```

>[!question]- Как эта модель выглядит "изнутри" — пример на сырых сокетах в проекте
> В `SocketBegin` есть пара классов, которая на голых `ServerSocket`/`ThreadPoolExecutor` (без Spring) воспроизводит именно то, что делает Tomcat + Spring MVC под капотом:
> - `org.example.server.TomcatLikeServer` — Acceptor-цикл (`serverSocket.accept()`) только принимает соединения и диспетчеризует каждое в `ThreadPoolExecutor` с параметрами `CorePoolSize/MaxPoolSize/QueueCapacity`, один в один как у `ThreadPoolTaskExecutor`. Обработка запроса (`handleRequest`) выполняется уже на потоке `http-exec-N` — аналог `DispatcherServlet → Controller → Service`.
> - Внутри — `RequestCountingService`, статический singleton-объект (как Spring-бин с default scope), к которому параллельно обращаются все `http-exec-N` потоки. Счётчик сделан на `AtomicInteger`, а не на обычном `int`-поле — именно чтобы не словить ту самую гонку данных, о которой калаут выше.
> - `org.example.client.TomcatLikeClient` — имитирует "наплыв пользователей": через `CountDownLatch` одновременно стартует несколько потоков-клиентов, каждый открывает своё TCP-соединение и шлёт "запрос", чтобы наглядно увидеть в логе сервера, как пул раздаёт конкурентные запросы по воркерам.
>
> Про сам сокет/TCP слой, на котором это всё построено (буферы, `accept()`, потоковая модель клиент-сервера) — см. [[TCP]].

# Tomcat, Servlet и DispatcherServlet

>[!question]- `DispatcherServlet` — это и есть Tomcat?
> Нет, это разные вещи на разных уровнях.
>
> **Tomcat** — сервлет-контейнер (реализация Servlet API), написан на чистой Java, отдельный от Spring проект (Apache). В Spring Boot это тот же самый Apache Tomcat, просто подключён как embedded-библиотека. Его работа — низкоуровневая: принимать TCP-соединения, парсить сырые HTTP-байты в `HttpServletRequest`/`HttpServletResponse`, управлять пулом воркеров (`http-nio-8080-exec-N`, `server.tomcat.threads.max`), вызывать зарегистрированный `Servlet`.
>
> **`DispatcherServlet`** — класс из `spring-webmvc`, тоже Java, но не имеет отношения к кодовой базе Tomcat. Это обычный `Servlet`, который Spring Boot регистрирует в Tomcat как единственный сервлет на паттерн `/`. Tomcat не знает про `@Controller`/`@RequestMapping` — весь роутинг по контроллерам целиком внутри `DispatcherServlet`.
>
> ```
> Tomcat (accept() + пул потоков + парсинг HTTP)
>     → находит зарегистрированный Servlet для пути "/"
>     → вызывает DispatcherServlet.service(request, response)
>         → HandlerMapping находит нужный @Controller-метод
>         → HandlerAdapter вызывает его
> ```
>
> В `org.example.server.TomcatLikeServer` из проекта: цикл `accept()` + `ThreadPoolExecutor` — аналог работы Tomcat; всё, что внутри `handleRequest()`, для простоты слито в одно — в реальности это два разных слоя (вызов Tomcat'ом сервлета + внутренний роутинг `DispatcherServlet`'а).

>[!question]- Что такое Servlet — тоже просто "обработчик запроса"?
> Да, но формализованный: Servlet — Java-класс, реализующий интерфейс `jakarta.servlet.Servlet` (обычно через `extends HttpServlet`), с методами жизненного цикла `init()` → `service()`/`doGet()`/`doPost()` → `destroy()`. Контейнер сам создаёт **один экземпляр** (синглтон, как и Spring-бин) и сам вызывает `service()` на каждый подходящий по URL-паттерну запрос, на своём worker-потоке.
>
> Метод `@Controller`-класса — обработчик запроса *по факту*, но не Servlet в строгом смысле: не реализует интерфейс `Servlet`, не регистрируется в Tomcat напрямую, вызывается через рефлексию самим `DispatcherServlet` (единственным настоящим Servlet'ом приложения).
>
> **Два независимых слоя маппинга:**
> ```
> Tomcat: URL → Servlet                          (грубо, обычно только "/" → DispatcherServlet)
>                 ↓
> DispatcherServlet: URL → @Controller-метод      (тонко, через HandlerMapping/аннотации)
> ```
> Правило: всякий Servlet — обработчик запроса, но не всякий обработчик запроса — Servlet.

>[!question]- Tomcat — "контейнер сервлЕТОВ" (мн. число), но в Spring Boot по сути один `DispatcherServlet`. Почему множественное число?
> Название описывает **архитектурную возможность** контейнера, а не то, сколько сервлетов реально зарегистрировано в конкретном приложении.
>
> **Историческая причина:** до Spring MVC в классических Java EE-приложениях было нормой регистрировать много сервлетов в одном вебаппе — `UserServlet` на `/user/*`, `OrderServlet` на `/order/*`, отдельный сервлет под каждую JSP-страницу, каждый со своим маппингом в `web.xml`. Именно под это Tomcat и проектировался: держать и управлять жизненным циклом произвольного числа сервлетов одновременно, маршрутизируя по URL-паттерну.
>
> **В Spring Boot дело часто не ограничивается одним `DispatcherServlet` и на практике.** Ничто не мешает зарегистрировать дополнительные сервлеты рядом через `ServletRegistrationBean`. Реальный встроенный пример — **H2 Console**: при `spring.h2.console.enabled=true` Spring Boot регистрирует **отдельный** сервлет (`org.h2.server.web.WebServlet`) на `/h2-console/*`, полностью независимо от `DispatcherServlet` на `/`. В Tomcat в этот момент реально работают два сервлета одновременно.
>
> **Почему не переписать H2 Console как `@Controller`:** H2 Console — целое самодостаточное мини-приложение (свои HTML/CSS/JS-страницы внутри jar, своя логика роутинга, своя сессия подключения к БД), написанное авторами H2 задолго до Spring Boot, специально в виде обычного `Servlet`, чтобы работать в любом контейнере. Регистрация через `ServletRegistrationBean` — одна строчка; переписывание всего этого как `@Controller`-методы — бессмысленное дублирование уже готового и рабочего кода.
>
> **Про производительность:** разница между "контейнер сразу вызывает `service()` голого сервлета" и "контейнер вызывает `DispatcherServlet.service()`, который сам ищет `@Controller`-метод через `HandlerMapping`" — несколько лишних вызовов/поиск по мапе, микросекунды. На фоне реальной работы (БД, сериализация, сеть) неощутимо — это не довод ни за, ни против такой схемы.
>
> **Ещё Tomcat может хостить несколько независимых веб-приложений (WAR)** на разных context-path (`/app1`, `/app2`), каждое со своим classloader'ом и своим набором сервлетов. Embedded-режим Spring Boot сворачивает это до одного вебаппа с одним `DispatcherServlet` — это самый частый сценарий использования, а не предел возможностей контейнера.

>[!question]- Что будет, если URL совпадает и у отдельного сервлета (`/h2-console/*`), и у `@Controller` внутри `DispatcherServlet`
> Сработает правило Servlet API "самый длинный совпадающий URL-паттерн побеждает" — на уровне самого Tomcat, ещё до всякого Spring MVC:
> 1. точное совпадение пути — высший приоритет;
> 2. самый длинный wildcard-префикс (`/h2-console/*` длиннее и специфичнее, чем `/`);
> 3. совпадение по расширению (`*.jsp`);
> 4. `/` — сервлет по умолчанию, ловит всё, что не подошло более специфичным паттернам.
>
> `DispatcherServlet` зарегистрирован на `/` — самый низкий приоритет. Значит запрос `GET /h2-console/login.jsp` Tomcat отправит **прямо в H2-сервлет**, а `DispatcherServlet.service()` для этого URL вообще не вызовется — весь внутренний Spring-роутинг (`HandlerMapping` → ваш `@Controller`) до этого пути просто не доходит.
>
> Ваш `@Controller`-метод на `/h2-console/**` при этом останется корректно зарегистрирован внутри `HandlerMapping`, **без единой ошибки при старте** — но окажется мёртвым кодом: реально вызвать его через настоящий HTTP-запрос через embedded Tomcat невозможно.
>
> **Отличие от `Ambiguous mapping`** (см. калаут выше в разделе URL-маппинга): там Spring сам проверяет свои внутренние `@Controller`-методы на конфликт и падает при старте с ошибкой. Здесь — два независимых, ничего не знающих друг о друге механизма маршрутизации (Tomcat-уровень и Spring-уровень), между которыми нет никакой взаимной валидации.
>
> **Практическое следствие:** коварный класс багов — юнит-тест контроллера (например, через `MockMvc`, который часто идёт в обход реального Tomcat) может показать, что метод "работает", а в реальном приложении через настоящий embedded-сервер он никогда не вызовется, потому что маршрутизация до него не доходит.

>[!question]- Какой контракт реализует `DispatcherServlet` и когда вызывается `destroy()`
> Цепочка наследования:
> ```
> Servlet (интерфейс, jakarta.servlet)
>   └─ GenericServlet (Servlet API — базовая реализация init/destroy/getServletConfig)
>        └─ HttpServlet (Servlet API — service(), doGet/doPost/doPut/doDelete)
>             └─ HttpServletBean (Spring — init-параметры сервлета → bean-свойства)
>                  └─ FrameworkServlet (Spring — поднимает WebApplicationContext, сводит все HTTP-методы в processRequest())
>                       └─ DispatcherServlet (Spring — HandlerMapping, HandlerAdapter, ViewResolver, роутинг по контроллерам)
> ```
>
> Жизненный цикл одного экземпляра:
> - `init()` — один раз, обычно **при старте приложения** (Spring Boot использует `load-on-startup`, не ленивую инициализацию на первый запрос);
> - `service()` — на **каждый** запрос, много раз;
> - `destroy()` — один раз, при **штатной остановке** контейнера (`ApplicationContext.close()`, обработанный `SIGTERM` и т.д.). Внутри `FrameworkServlet.destroy()` закрывает `WebApplicationContext` → триггерит `@PreDestroy`/`DisposableBean.destroy()` у всех бинов (закрыть пул потоков, соединения к БД и т.п.).
>
> **При зависании (дедлок, бесконечный цикл) `destroy()` не вызывается** — это не сигнал остановки. При жёстком `kill -9`/OOM-killer — тоже не вызывается, JVM не успевает выполнить вообще никакой код.

>[!question]- `destroy()` участвует в бизнес-логике?
> Нет. Это чисто инфраструктурный хук: не получает `request`/`response`, не знает, какие запросы обрабатывались — вызывается вне контекста конкретного пользовательского запроса, просто как сигнал "контейнер завершает работу, освободи ресурсы".
>
> Если в `@Service`/`@Repository` есть свой `@PreDestroy` (например, "доотправить накопленные в буфере события") — это выглядит близко к бизнес-логике, но категориально всё равно не обработка чьего-то запроса, а разовая операция при остановке приложения.

>[!question]- Когда `destroy()`/graceful shutdown реально нужен на практике
> Смысл не в том, что "сервер ещё нужен", а в том, чтобы **не оставить грязь снаружи процесса**, когда он умирает:
> - **Незавершённые запросы** — штатный shutdown сначала перестаёт принимать новые соединения, но даёт уже начатым запросам доработать (grace period), вместо обрыва запроса посреди транзакции.
> - **Ресурсы на стороне других систем** — пул соединений к БД при `destroy()` явно закрывает каждое соединение (TCP FIN); при `kill -9` сервер БД считает соединение живым до своего таймаута.
> - **Consumer'ы очередей (Kafka/RabbitMQ)** — graceful shutdown может явно покинуть consumer-группу, ребалансировка партиций происходит мгновенно; иначе брокер ждёт `session.timeout.ms` (10–30 сек), в течение которых часть партиций никем не обрабатывается.
> - **Буферизованные данные** — шанс форсированно сбросить накопленный в памяти буфер (метрики, batch-insert) перед смертью процесса.
> - **Rolling deploy в Kubernetes** — самый частый практический триггер `SIGTERM` в облачных системах. K8s ждёт `terminationGracePeriodSeconds`, прежде чем добить `SIGKILL`. Без graceful shutdown **каждый деплой** будет обрывать реальные пользовательские запросы в момент переключения.
>
> Важно: это не про "новая версия убивает данные старой" — старая версия сама аккуратно доделывает свою работу перед смертью, никакого прямого взаимодействия между версиями нет. Та же логика работает и без всякого деплоя — при обычном рестарте, scale-down, обслуживании ноды. Деплои и автоскейлинг просто самый частый повод, по которому в облаке вообще прилетает `SIGTERM`.

>[!question]- Что такое `ServletContextListener`/другие `*Listener`-интерфейсы
> Ещё один, третий контракт в спецификации Servlet API — но принципиально другого типа: он не обрабатывает и не перехватывает запросы, а просто **реагирует на события жизненного цикла всего приложения**.
>
> ```java
> public interface ServletContextListener extends EventListener {
>     default void contextInitialized(ServletContextEvent sce) {}
>     default void contextDestroyed(ServletContextEvent sce) {}
> }
> ```
> - **`contextInitialized`** — один раз, когда веб-приложение целиком поднимается (создаётся `ServletContext`), **до** инициализации любого отдельного `Servlet`.
> - **`contextDestroyed`** — один раз, при остановке, **после** того как все сервлеты уже прошли свой `destroy()`.
>
> **Чем отличается от `Filter`:** `Filter` активно участвует в обработке **каждого конкретного запроса** — стоит в цепочке, может модифицировать/блокировать. `ServletContextListener` вообще не видит отдельные запросы — чисто событийный колбэк ("приложение стартовало"/"приложение останавливается"), ближе к паттерну Observer, чем к цепочке обязанностей.
>
> **Чем отличается от `Servlet.init()`/`destroy()`:** у `Servlet` эти методы привязаны к **одному конкретному** зарегистрированному сервлету (узкий scope). У `ServletContextListener` — scope это **всё приложение целиком**, независимо от числа зарегистрированных сервлетов.
>
> **Родственные интерфейсы того же семейства** (регистрируются через `@WebListener` или `ServletListenerRegistrationBean`):
> - `HttpSessionListener` — `sessionCreated`/`sessionDestroyed` (создание/уничтожение HTTP-сессии конкретного пользователя);
> - `ServletRequestListener` — `requestInitialized`/`requestDestroyed` (начало/конец обработки отдельного запроса — но только для наблюдения, не для блокировки, в отличие от `Filter`);
> - `ServletContextAttributeListener`, `HttpSessionAttributeListener` — изменения атрибутов контекста/сессии.
>
> **Связь со Spring:** `ContextLoaderListener` — именно такой `ServletContextListener`, которым Spring в классических WAR-деплоях поднимает корневой `WebApplicationContext` при старте приложения.

>[!question]- Это ещё один способ ничего не сломать при закрытии приложения (как `Servlet.destroy()` и `@PreDestroy`)? Зачем он нужен, если те уже есть?
> Да, цель та же, но нужен он из-за двух причин: **порядка вызова** и **независимости от Spring**.
>
> **1. Строгий порядок при остановке — ключевое.** Servlet API задаёт чёткую последовательность:
> ```
> 1. Все Filter'ы и Servlet'ы уничтожаются (в порядке, обратном инициализации)
>    → тут же отрабатывает DispatcherServlet.destroy(), который закрывает
>      Spring ApplicationContext → тут же отрабатывают все @PreDestroy бинов
> 2. И ТОЛЬКО ПОСЛЕ ЭТОГО контейнер вызывает contextDestroyed()
> ```
> `contextDestroyed()` — самый **последний** хук во всей цепочке остановки, срабатывающий уже после того, как Spring (со всеми бинами и `DispatcherServlet`) полностью прекратил существование. Если нужно освободить что-то, что обязано пережить сам Spring-контекст до самого конца — `@PreDestroy` не годится, он срабатывает раньше, пока Spring ещё жив и разбирается со своими бинами.
>
> Симметрично на старте: `contextInitialized()` срабатывает **до** инициализации любого сервлета (в т.ч. до `DispatcherServlet`, до поднятия Spring-контекста) — самый первый хук, полезный для инфраструктуры, от которой зависят уже сами сервлеты.
>
> **2. Не завязан на Spring вообще.** Это чистый механизм Servlet API, работающий даже без Spring. Актуально для сторонних библиотек, поставляющих себя как обычный `Servlet`/`Filter` (та же H2 Console) — у них может быть свой `ServletContextListener`, никак не связанный с жизненным циклом Spring-бинов, а также для сырых ресурсов вне Spring IoC (JNDI, статический кеш, нативные библиотеки), которые в принципе не могут иметь `@PreDestroy`, так как не являются бинами.
>
> На практике в обычном Spring Boot-приложении свой `ServletContextListener` пишут редко — `@PreDestroy`/`DisposableBean` покрывают почти все случаи, и сам Spring внутри уже использует такой механизм (`ContextLoaderListener`). Но как самый внешний, независимый от Spring слой он остаётся для случаев, когда нужна гарантия "выполнится строго после всего остального", или когда участник вообще не часть Spring-контекста.

>[!question]- Происходит ли переподключение при handShake в вебсокете
>нет - TCP соединение остается, меняется протокол поверх него 
>Если на сервере зарегистрирован @ServerEndpoint/Endpoint для этого пути — Tomcat отвечает 101 Switching Protocols (последний "чисто HTTP" ответ в этом диалоге) и с этого момента переинтерпретирует уже существующее TCP-соединение: байты, летящие по нему дальше, читаются и пишутся уже не как  HTTP-заголовки/тело, а как WebSocket-фреймы (свой бинарный формат кадров). 
>
> Это верно для **прямого** подключения клиент↔Tomcat, где `101` шлёт именно Tomcat. Если между ними стоит reverse-proxy (nginx) — картина иная: это уже **два независимых** TCP-соединения (клиент↔nginx и nginx↔Tomcat), каждое со своим отдельным переключением на WebSocket-фреймы. Подробный разбор — см. [[NGINX]].

>[!question]- Кроме WebSocket, Tomcat/Jetty/Undertow умеют Server-Sent Events (SSE)?
> Да, и тут даже проще, чем с WebSocket — **SSE не отдельный протокол, а частный случай обычного HTTP**. Клиент делает обычный `GET`, сервер отвечает с `Content-Type: text/event-stream` и держит соединение открытым, дописывая новые события в тело ответа по мере появления. Никакого handshake/upgrade, как у WebSocket — это тот же HTTP/1.1 запрос-ответ, просто растянутый во времени. Поэтому для SSE не нужен отдельный контракт вроде `jakarta.websocket`.
>
> **Нюанс:** если бы соединение держалось обычным блокирующим `service()`, поток контейнера (`http-exec-N`) был бы занят на всё время жизни SSE-соединения (минуты/часы) — при множестве клиентов пул быстро исчерпался бы (та же проблема "поток на запрос").
>
> Решается через **Servlet 3.0+ Async API** (`request.startAsync()` / `AsyncContext`) — третий значимый контракт контейнера, отдельный и от базового `service()`, и от `jakarta.websocket`. Он отвязывает обработку запроса от конкретного потока: поток, принявший запрос, сразу возвращается в пул, а запись в открытый response происходит позже, из другого места, когда появляются новые данные.
>
> В Spring MVC это обёрнуто в `SseEmitter` (наследник `ResponseBodyEmitter`):
> ```java
> @GetMapping("/stream")
> public SseEmitter stream() {
>     SseEmitter emitter = new SseEmitter();
>     // где-то в другом потоке/колбэке: emitter.send("event data");
>     return emitter;
> }
> ```
>
> Ключевое отличие от WebSocket: SSE **однонаправленный** (только сервер → клиент), поверх обычного HTTP, с автопереподключением из коробки через браузерный `EventSource`. WebSocket — полный дуплекс со своим протоколом фреймов после отдельного handshake и отдельного контракта `jakarta.websocket`.


>[!question]- Spring Security — это часть Spring MVC или отдельный модуль?
> Отдельный модуль, не часть Spring MVC.
>
> - **Spring MVC** (`spring-webmvc`) — часть самого Spring Framework: `DispatcherServlet`, `@Controller`, `HandlerMapping`.
> - **Spring Security** — самостоятельный top-level проект в экосистеме Spring (как Spring Boot, Spring Data, Spring Cloud), со своим репозиторием и версионированием. Модули: `spring-security-core`, `spring-security-web`, `spring-security-config`.
>
> Видно и на уровне зависимостей: `spring-boot-starter-web` (Spring MVC) **не тянет** Spring Security — нужен отдельный `spring-boot-starter-security`.
>
> **Доказательство независимости:** Spring Security не требует Spring MVC для работы — она встраивается на уровне обычного `Filter` из Servlet API (см. калауты выше), а значит может защищать любое сервлетное приложение вообще без единого `@Controller` (чистое JSP, голые сервлеты). Плюс есть отдельный механизм — **method security** (`@PreAuthorize`, `@Secured`) через Spring AOP на уровне вызовов методов `@Service`, без всякой привязки к вебу.
>
> Связь между модулями — это **интеграция**, а не вложенность: Spring Security добавляет удобства специально под Spring MVC (`@AuthenticationPrincipal` как параметр метода контроллера, автопрокидка CSRF-токена в Thymeleaf), но фундаментально это два независимых модуля.


>[!question]- Spring Security работает внутри `DispatcherServlet`, то есть до попадания в контроллер?
> Не внутри — **до**, на отдельном, более раннем уровне контейнера. Механизм — ещё один контракт Servlet API: **`jakarta.servlet.Filter`**, отдельная сущность от `Servlet`. У фильтра есть метод `doFilter(request, response, chain)`, он оборачивает вызов сервлета снаружи и может:
> 1. посмотреть/изменить запрос **до** сервлета;
> 2. **не вызвать** `chain.doFilter(...)` вообще — тогда сервлет (`DispatcherServlet`) даже не запустится;
> 3. посмотреть/изменить ответ **после** того, как сервлет отработал.
>
> Spring Security регистрируется как один такой `Filter` — бин `springSecurityFilterChain` (через `DelegatingFilterProxy`), который сам внутри себя прогоняет запрос через цепочку внутренних security-фильтров (`SecurityContextPersistenceFilter`, `UsernamePasswordAuthenticationFilter`, `ExceptionTranslationFilter`, `AuthorizationFilter`/`FilterSecurityInterceptor` и т.д.).
>
> Реальный порядок прохождения запроса:
> ```
> Tomcat
>   → цепочка Filter'ов (в т.ч. springSecurityFilterChain)   ← Spring Security здесь
>   → DispatcherServlet.service()
>       → HandlerMapping → Controller → Service
> ```
> Если пользователь не аутентифицирован/не авторизован — `AuthorizationFilter` может прервать цепочку и сразу вернуть `401`/`403`/редирект, и **ни `DispatcherServlet`, ни код контроллера вообще не выполнятся**.

>[!question]- `Filter` (и Spring Security) — это тоже обрабатывает сам контейнер (Tomcat), или это надстройка Spring?
> `Filter` — не надстройка Spring поверх Tomcat, а **такая же часть спецификации Servlet API**, как и сам `Servlet`. Оба контракта (`jakarta.servlet.Servlet` и `jakarta.servlet.Filter`) реализует и оркестрирует один и тот же контейнер.
>
> Как это происходит технически:
> 1. При старте embedded Tomcat автонастройка Spring Security регистрирует фильтр `springSecurityFilterChain` через `FilterRegistrationBean` (или Boot сам подхватывает любой бин типа `Filter`) — регистрация идёт через тот же API контейнера, что и регистрация `DispatcherServlet`, на URL-паттерн `/*` ("для любого запроса").
> 2. Дальше уже сам **Tomcat**, а не Spring, на каждый входящий запрос читает список замапленных фильтров, строит `FilterChain` и сам вызывает `doFilter()` первого фильтра — именно контейнер решает "сначала прогнать через фильтры, потом отдать сервлету".
> 3. Spring Security просто пользуется этим штатным механизмом контейнера — её единственный `Filter` уже сам, средствами Spring, прогоняет запрос через свою внутреннюю цепочку security-фильтров.
>
> Правильная формулировка: не "Spring Security перехватывает запрос до Tomcat", а "**Tomcat сам вызывает Spring Security как часть своей собственной, штатной фильтр-машинерии**, ещё до того, как решит, какому сервлету отдать запрос".
>
> Полная картина уровней:
> ```
> Tomcat: accept() + пул потоков
>   → Filter chain (Tomcat вызывает каждый фильтр по URL-паттерну; здесь Spring Security)
>       → Servlet (Tomcat вызывает единственный DispatcherServlet)
>           → HandlerMapping → Controller → Service
> ```


# application.properties как установить правильные настройки пула потоков


>[!question]- Что конкретно нужно знать/сделать, чтобы определить настройки `application.properties` — сводный чек-лист
> Входные данные и мониторинг — не один и тот же тип пункта: первое собирается один раз до деплоя, второе — постоянный цикл проверки уже после.
>
> **Входные данные (один раз, до первого деплоя):**
> 1. **Машина** — конкретно число ядер CPU (напрямую в формулу `потоков ≈ ядра × (1 + ожидание/вычисление)`) и объём RAM (больше потоков/DB-соединений = больше памяти).
> 2. **Архитектура/downstream** — не просто "какая БД", а конкретно размер её собственного пула ([[HikariCP]]) — он задаёт жёсткий потолок сверху для `threads.max`, сколько бы ядер CPU ни было.
> 3. **Протоколы кроме HTTP** — WebSocket радикально меняет нужный `max-connections`/`accept-count` (долгоживущие соединения вместо коротких запрос-ответов), в отличие от `threads.max`, который они почти не трогают.
> 4. **Требования по времени обработки (SLA)** — задаёт допустимую задержку, а через неё — нужный запас потоков под пиковую нагрузку.
>
> **Обязательный шаг между входными данными и продакшеном — нагрузочное тестирование** (`JMeter`/`Gatling`/`k6`): прогнать ожидаемую нагрузку на реальных цифрах из пунктов 1-4, а не полагаться только на формулу на бумаге.
>
> **Мониторинг (ELK/Prometheus/Grafana) — не вход, а постоянная обратная связь после деплоя.** Он не помогает **выбрать** начальные цифры, а показывает, когда предположения из пунктов 1-4 оказались неверны в реальном трафике (растущая очередь `accept-count`, `tomcat.threads.busy` у потолка) — и тогда настройки подкручиваются заново. Это итеративный цикл, а не единоразовый набор факторов.


>[!question]- `application.properties` когда появляется?
> Не связано с Spring MVC — файл доступен в **любом** Spring Boot проекте, даже без единого веб-стартера.
>
> Механизм его чтения — часть **ядра Spring Boot** (`spring-boot` + `spring-boot-autoconfigure`), а не `spring-webmvc`. `SpringApplication` при старте сам сканирует `src/main/resources/` на предмет этого файла — независимо от подключённых стартеров.
>
> Проверяется легко: проект через Spring Initializr вообще без веб-зависимостей (только `spring-boot-starter`) всё равно получает пустой `application.properties` по умолчанию. Даже консольное приложение (например, Kafka-консьюмер без единого HTTP-эндпоинта) им пользуется — для логирования, кастомных свойств, датасорса и т.д.
>
> Что реально зависит от стартера — не сам файл, а **какие свойства в нём что-то значат**:
> - `server.tomcat.threads.max=200` — заработает, только если есть `spring-boot-starter-web` (иначе нет embedded-сервера, который бы это читал);
> - `spring.jpa.hibernate.ddl-auto=update` — только с `spring-boot-starter-data-jpa`;
> - `spring.task.execution.pool.core-size=5` — только если реально используется `@Async`/`TaskExecutor`.
>
> Файл и механизм его подхвата — универсальны для Spring Boot; набор осмысленных свойств внутри растёт с каждым добавленным стартером, потому что каждый регистрирует свою auto-configuration, которая эти свойства читает.

>[!question]- А чему обязан своим появлением `src/main/resources/messages/messages.properties` (именно в подпапке `messages/`)?
> Тут два разных слоя, и подпапка — НЕ дефолт Spring Boot, в отличие от `application.properties` выше.
>
> 1. **Сама роль файла** (i18n-сообщения/переводы) — отдельный от Spring MVC механизм ядра Spring: интерфейс `org.springframework.context.MessageSource`, часть `ApplicationContext` (`spring-context`), используется для локализованного текста (сообщения валидации, UI-лейблы).
> 2. **Дефолтное поведение Spring Boot** (`MessageSourceAutoConfiguration`) само по себе искало бы файл в **корне** classpath — `spring.messages.basename=messages` по умолчанию резолвится в `classpath:messages.properties`, БЕЗ подпапки.
> 3. **Подпапка `messages/` появляется только если в проекте есть свой `@Bean MessageSource`**, явно перекрывающий автоконфигурацию (она сама уступает через `@ConditionalOnMissingBean`). Пример из реального проекта (`strikerstat`, `I18nConfig`):
>    ```java
>    @Bean
>    public MessageSource messageSource() {
>        ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
>        messageSource.setBasenames("messages/messages");   // ← вот здесь задаётся подпапка
>        messageSource.setFallbackToSystemLocale(false);
>        messageSource.setDefaultEncoding("UTF-8");
>        messageSource.setUseCodeAsDefaultMessage(true);     // ключ не найден → вернуть сам код вместо исключения
>        return messageSource;
>    }
>    ```
> Это чистое архитектурное решение автора проекта (не засорять корень `resources/` файлами локализации), а не поведение Spring "из коробки".

>[!question]- что такое `server.tomcat.accept-count`
> Размер **backlog**'а — очереди уже установленных на уровне ОС TCP-соединений, которые ждут, когда коннектор Tomcat освободит рабочий поток, чтобы их обработать. Дефолт — **100**.
>
> Как это работает по шагам:
> 1. Клиент делает TCP `SYN` → ОС на уровне ядра (не Java-код!) сама завершает TCP-хендшейк и кладёт готовое соединение в очередь `accept()` — это стандартный сокет-примитив, тот же самый, который мы разбирали в `AsyncTcpServer`/[[TCP]] (параметр `backlog` у `new ServerSocket(port, backlog)`).
>    - **`SYN`** — от англ. **synchronize** ("синхронизировать"). Это не отдельный пакет какого-то особого типа, а один из служебных бит-флагов в заголовке TCP (наряду с `ACK`, `FIN`, `RST` и др.) — им клиент помечает самый первый пакет, которым просит открыть новое соединение. Полное трёхстороннее рукопожатие TCP: `SYN` (клиент: "хочу открыть соединение, вот мой стартовый номер байта") → `SYN-ACK` (сервер: "принял, вот мой стартовый номер, подтверди") → `ACK` (клиент подтверждает) — после этого шага соединение официально считается **установленным** и попадает в очередь `accept()`.
> 2. Если в этот момент у Tomcat есть свободный поток (не выбран весь `threads.max`) — соединение почти сразу вынимается из очереди, поток начинает его обрабатывать.
> 3. Если свободных потоков нет (все `threads.max` заняты) — новое, уже установленное соединение просто **ждёт своей очереди** здесь, в этом backlog'е, вместо немедленного отказа.
> 4. Если очередь тоже переполнена (пришло больше `accept-count` соединений, ожидающих поток) — новым TCP-подключениям ОС отвечает отказом на уровне сокета (обычно `connection refused`/`reset`, до того как запрос вообще увидит Java-код).
>
> **Важно (см. также калаут выше про "Как настраивать `server.tomcat.*`"):** это НЕ JDK-очередь задач вроде `QueueCapacity` у `ThreadPoolTaskExecutor` — там программный `BlockingQueue` внутри JVM, здесь же — низкоуровневый TCP-механизм, за который отвечает ядро ОС, а не Java. Именно поэтому увеличение `accept-count` не создаёт новых потоков и не решает нехватку потоков само по себе — оно лишь даёт больше времени/места пережить кратковременный всплеск запросов, прежде чем ОС начнёт отклонять новые подключения.
>
> Растущая, не спадающая очередь `accept-count` под устойчивой (не всплесковой) нагрузкой — сигнал, что потоков/ресурсов ниже по цепочке (например, [[HikariCP]]) реально не хватает, а не повод просто раздуть это число.

>[!question]- что такое `server.tomcat.max-connections`
> Потолок числа соединений, которые коннектор Tomcat вообще **активно принимает и обслуживает** одновременно (регистрирует в своём NIO `Poller`). Дефолт — **8192**.
>
> Как соотносится с `accept-count` (калаут выше) — это два соседних, но разных рубежа:
> 1. Пока число уже принятых (`accept()`'нутых) соединений **меньше** `max-connections` — Acceptor-поток Tomcat продолжает в цикле забирать новые готовые соединения из очереди ОС и регистрировать их в `Poller` для обработки.
> 2. Как только счётчик активных соединений **упирается** в `max-connections` — Acceptor-поток **перестаёт вызывать `accept()`** и временно замирает. Новые уже установленные (TCP-хендшейк прошёл) соединения при этом никуда не деваются — они копятся в том самом backlog'е ОС размером `accept-count`, просто их пока никто не забирает.
> 3. Как только одно из активных соединений закрывается (освобождается место ниже `max-connections`) — Acceptor снова начинает вызывать `accept()` и забирает соединения из backlog'а по новой.
> 4. Если и `max-connections` заняты, и `accept-count`-backlog переполнен одновременно — только тогда ОС начинает по-настоящему отказывать новым `SYN`-пакетам (`connection refused`/`reset`).
>
> То есть `max-connections` — это "сколько Tomcat реально готов держать в работе прямо сейчас", а `accept-count` — "сколько ещё может подождать в очереди, пока не появится место". Оба лимита считают именно **TCP-соединения**, а не HTTP-запросы — почему это критично для WebSocket (соединение живёт часами, а не миллисекундами одного запроса) — см. калаут выше про долгоживущие WS-соединения и рост `max-connections`/`accept-count` в 4 раза в реальном конфиге.

>[!question]- Частая путаница: `accept-count` — это число ПРИНЯТЫХ соединений?
> Нет, наоборот — несмотря на название:
> - **`max-connections`** = сколько соединений Tomcat реально держит **активными** (уже приняты, в работе).
> - **`accept-count`** = очередь **ожидающих** — тех, что ещё **не приняты**, потому что `max-connections` уже занят под завязку.
>
> То есть `accept-count` — это не "количество принятых", а размер очереди тех, кто ждёт своей очереди на приём.

>[!question]- Полный сценарий: `threads.max=200` + `accept-count=1000` + `max-connections=32768` — когда что происходит
> Разбор по шагам, включая частую ошибку.
>
> **Пока соединений меньше `max-connections`** (32768) — Acceptor успевает забирать их из ОС сразу, очередь `accept-count` реально пустует.
>
> **Ошибочный вывод:** "если подключений > 200 (`threads.max`), значит все потоки заняты". Это неверно, и это ключевое место. Открытое соединение **само по себе не занимает поток**. NIO-коннектор (дефолт в Spring Boot) через небольшую группу `Poller`-потоков (`select()`/`epoll`) следит сразу за тысячами зарегистрированных сокетов почти бесплатно (файловый дескриптор + немного памяти) и вытаскивает соединение из пула `threads.max` **только в момент реальной обработки** — пришёл запрос, идёт `service()`. Простаивающее keep-alive-соединение между запросами или WebSocket, ждущий push-уведомления, **не держит ни одного потока**.
>
> Поэтому легко иметь 30000 открытых, но в основном простаивающих WS-соединений при 2-3 реально занятых потоках в этот момент. Потоки исчерпываются не от числа **открытых** соединений, а от числа соединений, **прямо сейчас активно обрабатываемых** внутри `service()` — это функция времени обработки запроса и частоты их прихода, а не общего числа подключений.
>
> (На старом блокирующем BIO-коннекторе было бы иначе — там 1 соединение = 1 поток на всё время его жизни. Дефолтный embedded Tomcat в Spring Boot — именно NIO, специально чтобы разорвать эту связь.)
>
> **Жёсткий отказ новым подключениям** — отдельный, не связанный с потоками рубеж: `max-connections` (32768, реально принятые) + `accept-count` (1000, ждущие в очереди на приём) = **33768** суммарной "вместимости конвейера". Только после этого порога ОС начинает по-настоящему отклонять новые `SYN` (`connection refused`/`reset`).
>
> Итог: исчерпание пула потоков и отказ в TCP-соединении — два независимых лимита на разных уровнях, они не суммируются друг с другом и не гарантируют друг друга.

>[!question]- базовые настройки томкат в application.properties
> В `strikerstat/backend-java/src/main/resources/application.properties`:
> ```properties
> server.tomcat.threads.max=200              # = дефолт (200) — можно было не писать
> server.tomcat.threads.min-spare=10         # = дефолт (10) — можно было не писать
> server.tomcat.accept-count=1000            # ОТЛИЧАЕТСЯ от дефолта (100 → 1000)
> server.tomcat.max-connections=32768        # ОТЛИЧАЕТСЯ от дефолта (8192 → 32768)
> ```
> `threads.max`/`min-spare` тут чисто декоративны (совпадают с дефолтом). А вот `accept-count`/`max-connections` — реальная, осознанная настройка, увеличенная в 4 раза относительно дефолта. Вероятная причина — большое число одновременных **WebSocket**-соединений (судьи/зрители/OBS-оверлей на турнирах, см. [[WebSocketStomp]]).

>[!question]- Но ведь WebSocket — не HTTP-запрос, при чём тут `max-connections`/`accept-count`?
> Ключевой момент: `max-connections`/`accept-count` считают не HTTP-**запросы**, а живые **TCP-соединения**, которыми управляет коннектор Tomcat (`Poller` в NIO) — а мы уже разбирали в калауте про handshake, что апгрейд на WebSocket **не создаёт новое соединение**, а просто переключает протокол поверх того же самого TCP-сокета, который Tomcat изначально принял `accept()`'ом. Поэтому апгрейженный WS-сокет как был, так и остаётся одним из соединений, занимающих слот в `max-connections`.
>
> **Почему именно WebSocket непропорционально давит на этот лимит, а обычный HTTP — нет:**
> - Обычное HTTP keep-alive соединение **переиспользуется** для десятков коротких запросов подряд, а между ними может закрываться/возвращаться в пул — один и тот же слот в `max-connections` за время сессии пользователя "обслуживает" много разных запросов, оборот высокий.
> - WebSocket-соединение, наоборот, **держится открытым безвылазно на всю сессию** — часы напролёт для судьи, у которого весь день открыта вкладка с турниром — и всё это время занимает **один и тот же** слот в `max-connections`, даже если реально по нему ничего не передаётся (просто ждёт push-уведомление).
>
> Отсюда и `accept-count`: если много WS-клиентов держат соединения открытыми долго (слоты не освобождаются с обычной HTTP-скоростью оборота), а в какой-то момент прилетает всплеск **новых** попыток подключения (открытие турнирной страницы у всех разом, шторм переподключений после сетевого сбоя — см. `tryReconnect` в [[WebSocketStomp]]) — этим новым TCP-соединениям может не хватить свободных слотов сразу, и они на короткое время встают в очередь `accept-count`, вместо немедленного отказа. Увеличение обоих параметров — это запас прочности именно под такие всплески одновременных долгоживущих соединений, а не под обычный HTTP-трафик.

>[!question]- Относятся ли `threads.max`/`min-spare`/`accept-count`/`max-connections` к транспортному уровню OSI?
> Не все одинаково:
> - **`accept-count`** — да, буквально: параметр `backlog` у системного вызова `listen()`, реализован самим ядром ОС как часть TCP-стека (тот же `backlog`, что у `new ServerSocket(port, backlog)` в [[TCP]]). Чистый L4-механизм.
> - **`max-connections`** — пограничный случай: лимит накладывается на объекты транспортного уровня (TCP-соединения), но сам лимит — решение приложения (кода коннектора Tomcat), а не свойство протокола TCP. У TCP нет встроенного понятия "максимум соединений" — это прикладная политика поверх L4-ресурса.
> - **`threads.max`/`threads.min-spare`** — вообще не относятся к OSI ни на каком уровне. Это внутреннее управление конкурентностью JVM-процесса (сколько потоков ОС параллельно исполняют код) — не описывает передачу байт по сети, категориально вне модели OSI.

>[!question]- Как добавить метрики по `threads.max`/`min-spare`/`max-connections` и `accept-count`
> **`threads.*` и `max-connections`** — готовы из коробки через Micrometer (`spring-boot-starter-actuator` + `micrometer-registry-prometheus`):
> ```properties
> management.endpoints.web.exposure.include=health,metrics,prometheus
> management.metrics.export.prometheus.enabled=true
> ```
> Метрики: `tomcat.threads.busy` (сколько потоков занято сейчас — ключевая для отслеживания приближения к `threads.max`), `tomcat.threads.current`, `tomcat.threads.config.max`, `tomcat.connections.current`, `tomcat.connections.config.max`.
>
> **`accept-count` метриками Micrometer не показывается** — это глубина очереди на уровне ядра ОС, к которой у JVM/Tomcat нет доступа для интроспекции. Смотреть только через ОС: `ss -lt` — колонка `Recv-Q` на listening-сокете = текущее число соединений в очереди backlog'а, `Send-Q` = сконфигурированный лимит (сам `accept-count`).

>[!question]- Как эти метрики хранятся и где их смотреть
> Три отдельных места, метрики в каждом живут по-разному:
> 1. **В приложении (Micrometer)** — только в памяти JVM-процесса, ничего не сохраняется; при рестарте история пропадает.
> 2. **Эндпоинт `/actuator/prometheus`** — не хранилище, а снимок: `curl .../actuator/prometheus` отдаёт текущие значения текстом в момент запроса, истории не видно.
> 3. **Prometheus** (отдельный, постоянно работающий сервис) — по scrape-конфигу периодически (обычно раз в 15с) опрашивает `/actuator/prometheus` и **сохраняет** каждый опрос как точку временного ряда в свою TSDB на диске. Здесь появляется реальная история.
>
> **Где смотреть:**
> - `curl .../actuator/prometheus | grep tomcat_threads` — быстрая проверка без инфраструктуры, без истории;
> - **Prometheus UI** (обычно `:9090`) — ручные PromQL-запросы, простые графики, обычно для отладки;
> - **Grafana** (обычно `:3000`) — сюда смотрят постоянно: подключается к Prometheus как источник данных, строит дашборды с историей/порогами/алертами. Сама Grafana ничего не хранит, только запрашивает и рисует то, что уже лежит в Prometheus.
>
> **Нюанс:** у Prometheus по умолчанию retention (срок хранения локальной TSDB) — обычно 15 дней, дальше данные удаляются, если не подключено что-то для долгого хранения (Thanos/Mimir/VictoriaMetrics). Prometheus и Grafana не входят в Spring Boot — разворачиваются отдельно (Docker Compose/Kubernetes), с адресом `/actuator/prometheus` вашего приложения в scrape-конфиге Prometheus.

# Микросервисы

>[!question]- У каждого микросервиса свой embedded Tomcat, или несколько сервисов могут обслуживаться одним Tomcat? Как принято делать
> Общепринятый подход — **каждый микросервис получает свой собственный embedded Tomcat, в своём собственном JVM-процессе.** Это стандарт для микросервисной архитектуры, а не просто один из вариантов.
>
> ```
> Микросервис A → java -jar service-a.jar → свой JVM-процесс → свой embedded Tomcat → свой порт
> Микросервис B → java -jar service-b.jar → свой JVM-процесс → свой embedded Tomcat → свой порт
> ```
> Каждый Spring Boot JAR — самодостаточный исполняемый файл (`spring-boot-starter-web` тащит embedded Tomcat по умолчанию). Дальше это упаковывается в свой Docker-образ, деплоится как отдельный набор подов Kubernetes (или контейнеров в Docker Compose), с собственным числом реплик для масштабирования. Перед всем этим стоит reverse-proxy/API Gateway/K8s Ingress (та же роль, что у nginx — см. [[NGINX]]), маршрутизирующий внешний трафик к нужному сервису; сами сервисы при вызове друг друга используют обычные HTTP-запросы по сети (через service discovery — Eureka/Consul/K8s DNS), а не разделяют один процесс.
>
> **Контраст со "старой" моделью** (тот самый исторический "контейнер сервлетов" — см. калаут выше "Tomcat — контейнер сервлЕТОВ"): раньше был один отдельно стоящий Tomcat/JBoss/WebLogic-процесс, в который деплоили несколько WAR-файлов одновременно — разные "приложения", но один общий JVM, один heap, один пул потоков.
>
> **Почему микросервисы ушли от общего контейнера:**
> - **Изоляция/blast radius** — утечка памяти, зависание или падение JVM в одном сервисе не задевает остальные, если у каждого свой процесс. В общем Tomcat упавший WAR мог утянуть весь процесс со всеми остальными приложениями.
> - **Независимый деплой и масштабирование** — можно перезапустить/обновить/отмасштабировать один сервис, не трогая остальные. Передеплой одного WAR в общем Tomcat — операционно рискованная операция (classloader leaks), а масштабировать можно только весь процесс целиком.
> - **Независимые версии зависимостей** — у каждого сервиса свой classpath, версии Spring/Java/библиотек не конфликтуют с соседями.
> - **Учёт ресурсов** — с отдельными процессами тривиально видеть "этот сервис жрёт X CPU/Y памяти" — критично для K8s resource limits и автоскейлинга именно этого сервиса. В общем JVM всё смешано в одном heap.
> - **Естественно ложится на модель контейнеров/K8s** — один Pod = один процесс = один сервис, что и позволяет независимо шедулить/перезапускать/скейлить каждый.
>
> **Когда всё же встречается общий Tomcat с несколькими WAR** — скорее исключение и признак legacy: старые, не контейнеризированные on-prem инсталляции, либо намеренный выбор для очень маленьких, тесно связанных внутренних инструментов (по сути уже "модульный монолит", а не микросервисы — при явном выборе микросервисной архитектуры с общим Tomcat теряются почти все причины, ради которых микросервисы вообще выбирались). ^tomcat-per-microservice
