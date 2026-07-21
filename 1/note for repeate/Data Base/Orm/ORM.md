# ORM — паттерны и где JPA/Hibernate стоят среди них

>[!question]- Что такое ORM — одной фразой
> Object-Relational Mapping — общий **подход**, а не спецификация: представление строк/таблиц реляционной БД в виде объектов языка программирования. Не привязан к Java — есть ORM в Python (SQLAlchemy, Django ORM), Ruby (ActiveRecord), .NET (EF).
>
> Иерархия конкретно в Java-мире (см. также [[JPA]]):
> ```
> ORM (концепция)
>   └── JPA (Java-спецификация ORM, только контракт: аннотации, EntityManager, JPQL)
>         ├── Hibernate (реализация — самая популярная, есть и родное API: Session, HQL)
>         ├── EclipseLink (реализация)
>         └── OpenJPA (реализация)
> ```

>[!question]- JPA — единственная Java-спецификация ORM, или есть другие
> Нет, не единственная — просто сегодня фактически монопольная.
>
> - **JDO (Java Data Objects, JSR 12/243)** — отдельная, более старая спецификация ORM для Java. Ключевое отличие от JPA: JDO задумывался как **datastore-agnostic** — должен был работать не только с реляционными БД, но и с объектными/NoSQL-хранилищами. JPA же изначально заточен именно под реляционные БД. JDO так и не получил экосистему/тулинг уровня JPA (Hibernate когда-то поддерживал и JDO-режим тоже) и сейчас практически мёртв — встречается только в единичных legacy-проектах.
> - **EJB 2.x Entity Beans** — предшественник JPA внутри самого Java EE, до 2006 года. Считался неудачным (громоздкий, требовал наследования от служебных классов контейнера, XML-дескрипторы). JPA (вышел вместе с EJB 3.0) создавался именно как замена этому подходу.
>
> Итог: JPA — не "единственная теоретически возможная" спецификация, а единственная, которая реально прижилась. Остальные либо мертвы (EJB Entity Beans), либо нишевые (JDO).

## Два паттерна ORM: Data Mapper vs Active Record

>[!question]- JPA/Hibernate vs Active Record — в чём разница паттернов
> Это два разных архитектурных паттерна ORM (термины из книги Мартина Фаулера *PoEAA*), а не два конкурирующих фреймворка одного уровня.
>
> **Active Record** — объект **сам знает, как себя сохранять**. Модель = данные + доступ к БД в одном классе.
> ```ruby
> # Ruby on Rails
> user = User.find(1)
> user.name = "Igor"
> user.save        # объект сам себя сохраняет
> User.where(active: true).count
> ```
> Так же устроены Django ORM, Eloquent (Laravel), большинство "лёгких" ORM.
>
> **Data Mapper** — объект **ничего не знает о БД**. Между доменной моделью и БД стоит отдельный слой (mapper/repository), который и делает всю работу с персистентностью.
> ```java
> // JPA/Hibernate
> User user = entityManager.find(User.class, 1L);
> user.setName("Igor");
> // объект User не имеет метода save() — сохранение делает EntityManager/Repository
> userRepository.save(user);
> ```
> JPA/Hibernate реализуют именно **Data Mapper**: `@Entity`-класс — чистый носитель данных, вся логика загрузки/сохранения/кеширования вынесена в `EntityManager`/`Session`/репозитории.

>[!question]- Active Record точно нарушает SRP?
> Формально — да, почти всегда: класс отвечает **и за бизнес-данные/поведение домена, и за персистентность** (SQL/сохранение) одновременно. Это два разных повода для изменения (правило SRP «одна причина для изменения на класс») — меняется схема БД → трогаем класс; меняется бизнес-правило → тоже трогаем тот же класс.
>
> Но на практике это осознанный компромисс, а не «баг архитектуры»:
> - **Плюс Active Record:** меньше кода/абстракций, быстрее писать CRUD-heavy приложения (Rails, Laravel, простые Django-проекты), нагляднее для маленьких доменов.
> - **Минус:** тестировать бизнес-логику без БД сложнее (модель тянет за собой персистентность), сложнее менять схему хранения независимо от домена, класс разрастается.
> - **Data Mapper (JPA/Hibernate)** платит за соблюдение SRP большей инфраструктурной сложностью (EntityManager, репозитории, mapping-конфигурация), зато Entity можно тестировать как обычный POJO и менять способ хранения, не трогая домен.
>
> Вывод: Active Record нарушает SRP **по определению паттерна**, но это осознанный trade-off простоты за модульность — годится для небольших/некомплексных доменов, плохо масштабируется на сложную бизнес-логику. См. также обсуждение Anemic vs Rich Domain Model в [[JPA]] (раздел «Подходы к работе») — это тот же вопрос «где живёт бизнес-логика относительно Entity».

## Граница JPA-спецификации и Hibernate-реализации

>[!question]- jpql к чему относится в хайбернате — часть спеки JPA или Hibernate-фича
> **JPQL — часть спецификации JPA**, не изобретение Hibernate. Любая JPA-реализация (Hibernate, EclipseLink, OpenJPA) обязана поддерживать JPQL и понимать одинаковый синтаксис.
>
> Hibernate при этом имеет собственный **HQL (Hibernate Query Language)** — язык, который **исторически появился раньше JPQL** и является его надмножеством: весь JPQL-код — валидный HQL, но не наоборот (у HQL есть свои расширения: например, `insert into ... select`, некоторые функции, обращение к Hibernate-специфичным возможностям).
>
> На практике когда пишешь `@Query("SELECT ...")` в Spring Data JPA — это JPQL (переносимо между провайдерами). Если используешь `session.createQuery(...)` через нативный Hibernate `Session` — это уже HQL-контекст (хотя синтаксически в 95% случаев неотличимо от JPQL).

>[!question]- Пример на Java: JPQL vs HQL
> Простой SELECT синтаксически одинаков и работает через оба входа — это и есть смысл фразы "JPQL — подмножество HQL":
> ```java
> // JPQL — через EntityManager (JPA API, стандарт, переносим между Hibernate/EclipseLink/OpenJPA)
> @Query("SELECT e FROM Event e WHERE e.eventDate > :date")
> List<Event> findUpcoming(@Param("date") LocalDate date);
>
> // то же самое напрямую через EntityManager
> TypedQuery<Event> query = entityManager.createQuery(
>     "SELECT e FROM Event e WHERE e.eventDate > :date", Event.class);
> query.setParameter("date", LocalDate.now());
> List<Event> events = query.getResultList();
> ```
> А вот HQL-расширение, которого в спецификации JPA нет вообще — bulk `insert into ... select`. Через `EntityManager`/JPQL так не сделать, только через нативный Hibernate `Session`:
> ```java
> Session session = entityManager.unwrap(Session.class); // выход из JPA в родное Hibernate API
>
> String hql = "insert into ArchivedEvent (id, name, archivedDate) " +
>              "select e.id, e.name, current_date() from Event e where e.eventDate < :cutoff";
>
> int inserted = session.createMutationQuery(hql)
>         .setParameter("cutoff", LocalDate.now().minusYears(1))
>         .executeUpdate();
> ```
> `entityManager.createQuery(hql, ...)` с таким запросом просто не скомпилируется по контракту JPA — `insert ... select` не входит в грамматику JPQL. Это ровно тот случай, когда HQL "шире" JPQL: возможность есть только у Hibernate-реализации, и код перестаёт быть переносимым на другой JPA-провайдер. (см. также [[Proxy Object]] про похожий вопрос "кто обрабатывает — Spring или Hibernate"):
>
> | Что | Кому принадлежит |
> |---|---|
> | JPQL, `@Entity`, `EntityManager`, Criteria API | JPA-спецификация |
> | HQL, `Session`, HQL-расширения, механизм прокси/lazy loading как таковой | Hibernate-реализация (сверх контракта JPA) |
