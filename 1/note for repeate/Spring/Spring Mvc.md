# Spring MVC

>[!question]- Что такое Spring MVC?
> Spring MVC — модуль `spring-webmvc`, реализующий паттерн **Model-View-Controller** для HTTP-приложений. Подтягивается через `spring-boot-starter-web` (включает встроенный Tomcat, Jackson, валидацию).
> Центральный класс — **`DispatcherServlet`**: единая точка входа для HTTP-запросов, распределяет их по контроллерам, использует `HandlerMapping` (по аннотациям), `HandlerAdapter`, `ViewResolver` и т.д.

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