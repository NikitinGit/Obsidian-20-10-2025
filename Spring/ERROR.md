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








