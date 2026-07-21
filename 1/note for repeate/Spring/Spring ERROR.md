
>[!question]- Ошибка расположения бинов
>2025-06-19T13:43:20.516+05:00  WARN 26031 --- [  restartedMain] ConfigServletWebServerApplicationContext : Exception encountered during context initialization - cancelling refresh attempt: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'openEventController': Unsatisfied dependency expressed through field 'openEventService': Error creating bean with name 'openEventService': Unsatisfied dependency expressed through field 'openEventBattleService': Error creating bean with name 'openEventBattleService': Unsatisfied dependency expressed through field 'drawPreparationService': Error creating bean with name 'drawPreparationService': Unsatisfied dependency expressed through field 'openEventBattleService': Error creating bean with name 'openEventBattleService': Requested bean is currently in creation: Is there an unresolvable circular reference?
2025-06-19T13:43:20.518+05:00  INFO 26031 --- [  restartedMain] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
2025-06-19T13:43:20.519+05:00 TRACE 26031 --- [  restartedMain] o.h.type.spi.TypeConfiguration$Scope     : Handling #sessionFactoryClosed from [org.hibernate.internal.SessionFactoryImpl@f5bb38d] for TypeConfiguration
2025-06-19T13:43:20.519+05:00 DEBUG 26031 --- [  restartedMain] o.h.type.spi.TypeConfiguration$Scope     : Un-scoping TypeConfiguration [org.hibernate.type.spi.TypeConfiguration$Scope@58c495bd] from SessionFactory [org.hibernate.internal.SessionFactoryImpl@f5bb38d]
2025-06-19T13:43:20.519+05:00  INFO 26031 --- [  restartedMain] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
2025-06-19T13:43:20.523+05:00  INFO 26031 --- [  restartedMain] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.
2025-06-19T13:43:20.525+05:00  INFO 26031 --- [  restartedMain] o.apache.catalina.core.StandardService   : Stopping service [Tomcat]
2025-06-19T13:43:20.533+05:00  INFO 26031 --- [  restartedMain] .s.b.a.l.ConditionEvaluationReportLogger : 

>[!question]- Ошибка в методе репозитория, когда не используется @Param
>java.lang.IllegalStateException: For queries with named parameters you need to provide names for method parameters; Use @Param for query method parameters, or when on Java 8+ use the javac flag -parameters

>[!question]- Ошибка удаления связанных записей таблиц
>Cannot delete or update a parent row: a foreign key constraint fails (`co34818_sign`.`judges_scores`, CONSTRAINT
>`judges_scores_battle_id_foreign` FOREIGN KEY (`battle_id`) REFERENCES `battles` (`battle_id`))
>2025-07-07T11:32:39.652+03:00  INFO 126006 --- [nio-6300-exec-6] c.s.w.config.RequestLoggingInterceptor   : Request durationfor? 
>POST /api/open_events/organizer/delete_battles 7.086 s
> cascade = CascadeType.ALL, orphanRemoval = true работает только при ОРМ удалении
> battlesRepository.deleteBattlesByBattleIds(battleIdList); // Нативный запрос → каскады игнорируются

>[!question]- Ambiguous mapping. Cannot map ... to {GET [/api/test/3]}
> **Лог:**
> ```
> org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'requestMappingHandlerMapping'
>   ... Ambiguous mapping. Cannot map 'aspectController' method
>   com.example.testlinux.controller.AspectController#endPointN3()
>   to {GET [/api/test/3]}: There is already 'aspectController' bean method
>   com.example.testlinux.controller.AspectController#endPointParamN1(Integer, Integer) mapped.
> Caused by: java.lang.IllegalStateException: Ambiguous mapping. ...
>   at ...RequestMappingHandlerMapping...
> ```
>
> **Причина:** два метода контроллера зарегистрированы на **одинаковый URL + HTTP-метод**. Spring не может выбрать, какой вызывать → контекст не поднимается, приложение падает на старте.
> ```java
> @GetMapping("/3")
> public ResponseEntity<String> endPointParamN1(@RequestParam Integer eventId,
>                                               @RequestParam Integer userId) { ... }
>
> @GetMapping("/3")     // ← тот же путь, тот же GET
> public ResponseEntity<String> endPointN3() { ... }
> ```
>
> **Важно:** `@RequestParam` **НЕ участвует в URL-маппинге** — query string (то, что после `?`) не часть пути. Поэтому `/3?eventId=1` и просто `/3` для Spring это **один и тот же маппинг**. Различить методы по наличию query-параметров нельзя.
>
> **Решение:** дать одному из методов другой путь:
> ```java
> @GetMapping("/param")
> public ResponseEntity<String> endPointParamN1(@RequestParam Integer eventId,
>                                               @RequestParam Integer userId) { ... }
>
> @GetMapping("/3")
> public ResponseEntity<String> endPointN3() { ... }
> ```
>
> **Что может ещё вызывать `Ambiguous mapping`:**
> - Одинаковый путь + одинаковый HTTP-method.
> - Одинаковый путь с разными path-variable именами (`/{id}` и `/{userId}` — для Spring это идентичные паттерны).
> - Класс с `@RequestMapping("/api")` + метод с `@GetMapping("/x")`, в другом классе уже есть `/api/x`.
>
> **Как НЕ путать с конфликтом по query-параметрам:** различать по `?param=...` можно через атрибут `params`:
> ```java
> @GetMapping(value = "/3", params = "eventId")    // только если ?eventId=... есть
> @GetMapping(value = "/3", params = "!eventId")   // только если ?eventId=... НЕТ
> ```
> Но это редкий случай — обычно проще развести по разным путям.

>[!question]- AopUtils
> в нем находятся ошибки и чаще всего вызываются при ошибках в таблицах , отсутствие таблицы в БД 





