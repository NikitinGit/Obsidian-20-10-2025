
2026-06-28T13:36:00.424+05:00  INFO 7166 --- [SpringSecurityDomo] [           main] o.e.s.SpringSecurityDomoApplication      : Starting SpringSecurityDomoApplication using Java 21.0.12 with PID 7166 (/home/igor/IdeaProjects/SpringSecurityDomo/target/classes started by igor in /home/igor/IdeaProjects/SpringSecurityDomo)
2026-06-28T13:36:00.425+05:00  INFO 7166 --- [SpringSecurityDomo] [           main] o.e.s.SpringSecurityDomoApplication      : No active profile set, falling back to 1 default profile: "default"
2026-06-28T13:36:00.626+05:00  INFO 7166 --- [SpringSecurityDomo] [           main] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data JPA repositories in DEFAULT mode.
2026-06-28T13:36:00.632+05:00  INFO 7166 --- [SpringSecurityDomo] [           main] .s.d.r.c.RepositoryConfigurationDelegate : Finished Spring Data repository scanning in 4 ms. Found 0 JPA repository interfaces.
2026-06-28T13:36:00.791+05:00  INFO 7166 --- [SpringSecurityDomo] [           main] o.s.boot.tomcat.TomcatWebServer          : Tomcat initialized with port 8080 (http)
2026-06-28T13:36:00.797+05:00  INFO 7166 --- [SpringSecurityDomo] [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2026-06-28T13:36:00.797+05:00  INFO 7166 --- [SpringSecurityDomo] [           main] o.apache.catalina.core.StandardEngine    : Starting Servlet engine: [Apache Tomcat/11.0.22]
2026-06-28T13:36:00.809+05:00  INFO 7166 --- [SpringSecurityDomo] [           main] b.w.c.s.WebApplicationContextInitializer : Root WebApplicationContext: initialization completed in 368 ms
2026-06-28T13:36:00.874+05:00  INFO 7166 --- [SpringSecurityDomo] [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2026-06-28T13:36:01.316+05:00  INFO 7166 --- [SpringSecurityDomo] [           main] com.zaxxer.hikari.pool.HikariPool        : HikariPool-1 - Added connection com.mysql.cj.jdbc.ConnectionImpl@9408857
2026-06-28T13:36:01.317+05:00  INFO 7166 --- [SpringSecurityDomo] [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
2026-06-28T13:36:01.329+05:00  INFO 7166 --- [SpringSecurityDomo] [           main] org.hibernate.orm.jpa                    : HHH008540: Processing PersistenceUnitInfo [name: default]
2026-06-28T13:36:01.358+05:00  INFO 7166 --- [SpringSecurityDomo] [           main] org.hibernate.orm.core                   : HHH000001: Hibernate ORM core version 7.4.1.Final
2026-06-28T13:36:01.537+05:00  INFO 7166 --- [SpringSecurityDomo] [           main] o.s.o.j.p.SpringPersistenceUnitInfo      : No LoadTimeWeaver setup: ignoring JPA class transformer
2026-06-28T13:36:01.568+05:00  WARN 7166 --- [SpringSecurityDomo] [           main] org.hibernate.orm.deprecation            : HHH90000025: MySQLDialect does not need to be specified explicitly using 'hibernate.dialect' (remove the property setting and it will be selected by default)
2026-06-28T13:36:01.574+05:00  INFO 7166 --- [SpringSecurityDomo] [           main] org.hibernate.orm.connections.pooling    : HHH10001005: Database info:
	Database JDBC URL [jdbc:mysql://localhost:3306/strikerstat_local?serverTimezone=UTC&useLegacyDatetimeCode=false]
	Database driver: MySQL Connector/J
	Database dialect: MySQLDialect
	Database version: 8.0.46
	Default catalog/schema: strikerstat_local/undefined
	Autocommit mode: undefined/unknown
	Isolation level: REPEATABLE_READ [default REPEATABLE_READ]
	JDBC fetch size: none
	Pool: DataSourceConnectionProvider
	Minimum pool size: undefined/unknown
	Maximum pool size: undefined/unknown
2026-06-28T13:36:01.706+05:00  INFO 7166 --- [SpringSecurityDomo] [           main] org.hibernate.orm.core                   : HHH000489: No JTA platform available (set 'hibernate.transaction.jta.platform' to enable JTA platform integration)
2026-06-28T13:36:01.709+05:00  INFO 7166 --- [SpringSecurityDomo] [           main] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
2026-06-28T13:36:01.734+05:00  WARN 7166 --- [SpringSecurityDomo] [           main] JpaBaseConfiguration$JpaWebConfiguration : spring.jpa.open-in-view is enabled by default. Therefore, database queries may be performed during view rendering. Explicitly configure spring.jpa.open-in-view to disable this warning
2026-06-28T13:36:01.763+05:00  WARN 7166 --- [SpringSecurityDomo] [           main] .s.a.UserDetailsServiceAutoConfiguration : 

Using generated security password: b00a4545-33b7-46cc-bb60-34afc99e70d6

This generated password is for development use only. Your security configuration must be updated before running your application in production.

2026-06-28T13:36:01.766+05:00  INFO 7166 --- [SpringSecurityDomo] [           main] r$InitializeUserDetailsManagerConfigurer : Global AuthenticationManager configured with UserDetailsService bean with name inMemoryUserDetailsManager
2026-06-28T13:36:01.932+05:00  INFO 7166 --- [SpringSecurityDomo] [           main] o.s.boot.tomcat.TomcatWebServer          : Tomcat started on port 8080 (http) with context path '/'
2026-06-28T13:36:01.934+05:00  INFO 7166 --- [SpringSecurityDomo] [           main] o.e.s.SpringSecurityDomoApplication      : Started SpringSecurityDomoApplication in 1.638 seconds (process running for 1.83)

