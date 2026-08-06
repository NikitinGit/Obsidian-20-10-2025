# Задачи

1. [ ] propagation — открывать новую транзакцию или использовать текущую — ⚠️ ответ потерян, задача снова открыта
2. [x] equals/hashcode в @Entity — см. [[JPA equals hashCode]]
3. [ ] [[SPRING ЛОГИ ЗАПУСКА]] — разобрать
4. [ ] @EqualsAndHashCode.Exclude / @ToString.Exclude — проверить код на бесконечную рекурсию, неверное сравнение объектов, N+1 — action item, см. [[JPA equals hashCode]]
5. [ ] обязательно ли указывать в hibernate timezone, как принято хранить время с часовым поясом — открыто, памятка не написана
6. [ ] EntityManager.detach — зачем нужен, другие методы общая теория persistence context в [[Proxy Object]]
7. [x] статический прокси vs динамический — см. [[Proxy Object]]
8. [ ] интерсептор и мидлвеар — частично см. [[Spring AOP]] (HandlerInterceptor), дописать сравнение
9. [x] фабричный метод паттерн, почему @Bean — это он — см. [[IoC Spring]]
10. [x] проблемы Hibernate-прокси: equals/hashcode не работает — см. [[JPA equals hashCode]]
11. [ ] безопасно ли сравнивать по entity вместо id при выборке (`findAllByBattleAndJudge(Battle, Judge)`) — ⚠️ ответ потерян, задача снова открыта
12. [ ] почему в JudgeScoreRepository можно делать выборку чужих сущностей и правильно ли так делать — открыто, см. [[JPA]] (Custom Repository)
13. [ ] прочитать https://docs.spring.io/spring-framework/reference/web.html и сравнить со своим проектом
14. [x] QueryDSL — см. [[JPA]]
15. [ ] как кэшировать GET-запросы
16. [ ] всегда ли есть смысл делать связь на уровне Entity, а не только в БД — открыто, рядом с [[JPA N+1 problem]]
17. [ ] какие аннотации знать на собеседовании — см. также «Собеседование/ВОПРОСЫ ПО СОБЕЗУ»
18. [ ] @Profile — открыто, дом [[IoC Spring]]
19. [ ] @ConditionalOnProperty — открыто, дом [[IoC Spring]]
20. [x] кэш первого и второго уровня — см. [[Кэши JPA-Hibernate]]
21. [ ] можно ли подключиться к БД без application.properties
22. [ ] как запустить не-DDL миграции — см. MigrationService.java, рядом «Миграции»
23. [x] встроенные серверы Tomcat/Jetty — см. [[Spring Mvc]]
24. [x] отличие @GetMapping от @RequestMapping(method=GET) — см. [[Spring Mvc]]
25. [x] AOP в Spring — см. [[Spring AOP]]
26. [ ] как создаётся изолированный контекст — частично см. [[IoC Spring]] (несколько IoC-контейнеров)
27. [x] hikary pool — см. [[HikariCP]]
28. [ ] подключения к БД: credentials, pool settings — частично [[HikariCP]], дописать про credentials
29. [x] елк стек, актуатор, графана — см. Мониторинг/

Удалены как дубли при актуализации: «связи в Entity vs без связей» (уже первый пункт в [[JPA N+1 problem]]), «Domain-Driven Design» (свой файл «Domen driven design»), фрагмент про банки/страхование (уже внутри ответа «предназначение» ниже).

# Основы 
>[!question]- запустить спринг проект
>mvn spring-boot:run
# Остальное 

>[!question]- если @Transactionl не используется то в каких методах не нужен .save
>```List<Battle> battles = battlesRepository.getBattlesFromClient(battleIdList); 
>for  (Battle battle : battles) {  
 >   battlesRepository.delete(battle);  
>}
>```

Два механизма в Spring/Hibernate                                                                                                                                                                                                                                                                                  
   
  Spring AOP (прокси) — обрабатывает Spring:                                                                                                                                                                                                                                                                        
  - @Transactional
  - @Cacheable
  - @Async
  - @PreAuthorize

  Создаёт прокси-обёртку вокруг бина, перехватывает вызовы методов.

  Hibernate маппинг — обрабатывает Hibernate напрямую:
  - @Convert
  - @Enumerated
  - @Column
  - @OneToMany

  Никакого прокси — просто метаданные для ORM при работе с JDBC.

  ---
  Как понять кто обрабатывает?

  Простой ориентир:

  ┌──────────────────────────────────────────┬─────────────────────────┐
  │                  Вопрос                  │          Ответ          │
  ├──────────────────────────────────────────┼─────────────────────────┤
  │ Аннотация на методе/классе бина?         │ скорее всего Spring AOP │
  ├──────────────────────────────────────────┼─────────────────────────┤
  │ Аннотация на поле Entity?                │ скорее всего Hibernate  │
  ├──────────────────────────────────────────┼─────────────────────────┤
  │ Аннотация из пакета org.springframework? │ Spring                  │
  ├──────────────────────────────────────────┼─────────────────────────┤
  │ Аннотация из пакета jakarta.persistence? │ JPA/Hibernate           │
  └──────────────────────────────────────────┴─────────────────────────┘



