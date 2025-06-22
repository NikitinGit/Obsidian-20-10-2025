>[!question]- Ошибка расположения бинов
>2025-06-19T13:43:20.516+05:00  WARN 26031 --- [  restartedMain] ConfigServletWebServerApplicationContext : Exception encountered during context initialization - cancelling refresh attempt: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'openEventController': Unsatisfied dependency expressed through field 'openEventService': Error creating bean with name 'openEventService': Unsatisfied dependency expressed through field 'openEventBattleService': Error creating bean with name 'openEventBattleService': Unsatisfied dependency expressed through field 'drawPreparationService': Error creating bean with name 'drawPreparationService': Unsatisfied dependency expressed through field 'openEventBattleService': Error creating bean with name 'openEventBattleService': Requested bean is currently in creation: Is there an unresolvable circular reference?
2025-06-19T13:43:20.518+05:00  INFO 26031 --- [  restartedMain] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
2025-06-19T13:43:20.519+05:00 TRACE 26031 --- [  restartedMain] o.h.type.spi.TypeConfiguration$Scope     : Handling #sessionFactoryClosed from [org.hibernate.internal.SessionFactoryImpl@f5bb38d] for TypeConfiguration
2025-06-19T13:43:20.519+05:00 DEBUG 26031 --- [  restartedMain] o.h.type.spi.TypeConfiguration$Scope     : Un-scoping TypeConfiguration [org.hibernate.type.spi.TypeConfiguration$Scope@58c495bd] from SessionFactory [org.hibernate.internal.SessionFactoryImpl@f5bb38d]
2025-06-19T13:43:20.519+05:00  INFO 26031 --- [  restartedMain] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
2025-06-19T13:43:20.523+05:00  INFO 26031 --- [  restartedMain] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.
2025-06-19T13:43:20.525+05:00  INFO 26031 --- [  restartedMain] o.apache.catalina.core.StandardService   : Stopping service [Tomcat]
2025-06-19T13:43:20.533+05:00  INFO 26031 --- [  restartedMain] .s.b.a.l.ConditionEvaluationReportLogger : 



