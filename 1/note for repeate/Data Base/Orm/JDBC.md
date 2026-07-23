1. [x] JDBC почему называется Connectivity - занимается только соединеним к БД ? select, update, insert, delete, trancute и остальные команды sql  на нем не работают?
2. [ ] что содержит пакет jakarta - jdbc/ hibernate/jpa ?
3. [ ] "В Spring `@Transactional`-методе физическое соединение "прикрепляется" к транзакции при **первом** обращении к БД внутри неё и удерживается до `commit`/`rollback` — все запросы в рамках одной транзакции идут через одно и то же соединение." - ОЗНАЧАЕТ ЛИ ЭТО ЧТО ДЛЯ КАЖДОГО `@Transactional` СОЗДАЕТСЯ НОВОЕ СЕДИНЕНИЕ И ПОСЛЕ ЗАВЕРШЕНИЯ ТРАНЗАКЦИИ ОНО ЗАКРЫВАЕТСЯ АВТОМАТИЧЕСКИ ?
4. [ ] Как в линкс получить только те записи в логах, у котторых poolConnection > 8 
5. [ ] 2026-07-22T12:46:36.281+05:00  INFO 10407 --- [        async-1] i.StatisticalLoggingSessionEventListener : Session Metrics {
    365160 nanoseconds spent acquiring 1 JDBC connections;
    0 nanoseconds spent releasing 0 JDBC connections;
    45809 nanoseconds spent preparing 1 JDBC statements;
    5735218 nanoseconds spent executing 1 JDBC statements;
    0 nanoseconds spent executing 0 JDBC batches;
    0 nanoseconds spent performing 0 L2C puts;
    0 nanoseconds spent performing 0 L2C hits;
    0 nanoseconds spent performing 0 L2C misses;
    0 nanoseconds spent executing 0 flushes (flushing a total of 0 entities and 0 collections);
    4393 nanoseconds spent executing 1 partial-flushes (flushing a total of 0 entities and 0 collections)
}
# JDBC — Connection, Statement, ResultSet

>[!question]- Что такое JDBC — одной фразой
> **Java Database Connectivity** — стандартный низкоуровневый Java API для работы с реляционными БД: открыть соединение, отправить SQL, прочитать результат. Hibernate (и любая JPA-реализация) **не заменяет** JDBC, а строится поверх него — в итоге любой JPQL/HQL-запрос транслируется Hibernate в JDBC-вызовы.

>[!question]- Почему называется "Connectivity"?
> Название не означает "только установка соединения" — это широкий смысл слова "connectivity" ("связность"), а не буквально "connection management".
>
> JDBC создавался как Java-аналог **ODBC (Open Database Connectivity)** — стандарта от Microsoft начала 90-х для C/Windows. Название скопировано по той же схеме ("Java" + "Database Connectivity"), чтобы показать: это тот же тип решения, только для Java.
>
> "Connectivity" здесь — как "сетевая связность" (network connectivity): не означает только "воткнут кабель", а всю способность передавать/принимать данные по сети. Смысловой акцент — на том, что JDBC даёт **единый API поверх разных вендоров БД** (MySQL, PostgreSQL, Oracle...): приложение "связано" с любой БД одинаковым кодом независимо от того, какой драйвер подставлен под капотом. Это про универсальность моста Java ↔ БД, а не про то, что API умеет только открывать соединение.
>
> `Connection` — лишь точка входа. SQL-команды любого типа (DML: `SELECT`/`INSERT`/`UPDATE`/`DELETE`, DDL: `TRUNCATE`/`CREATE`/`ALTER`, вызовы хранимых процедур) выполняются через `Statement`/`PreparedStatement`/`CallableStatement` — это такая же неотъемлемая часть JDBC, просто название исторически унаследовано от ODBC и описывает API целиком, а не один его класс.
## Connection

>[!question]- Что такое `Connection`
> Представляет **одну сессию** с базой данных — физическое TCP-соединение + аутентифицированный контекст на стороне БД. Через `Connection` создаются `Statement`/`PreparedStatement`, на нём же держится состояние транзакции (`autoCommit`, isolation level).
> ```java
> Connection connection = dataSource.getConnection();
> connection.setAutoCommit(false); // начало ручной транзакции
> ```
> `Connection` — самый "дорогой" из трёх объектов: его создание — это реальное сетевое рукопожатие + аутентификация на БД, отсюда и весь смысл connection pooling (см. раздел DataSource ниже).

## Statement / PreparedStatement

>[!question]- Что такое `Statement` и чем `PreparedStatement` от него отличается
> `Statement` — объект, представляющий **одну SQL-команду**, отправляемую через `Connection`.
> ```java
> Statement st = connection.createStatement();
> ResultSet rs = st.executeQuery("SELECT * FROM events WHERE id = " + eventId); // ❌ конкатенация — SQL injection
> ```
> `PreparedStatement` — тот же `Statement`, но **предкомпилированный**, с `?`-плейсхолдерами вместо конкатенации:
> ```java
> PreparedStatement ps = connection.prepareStatement("SELECT * FROM events WHERE id = ?");
> ps.setInt(1, eventId);
> ResultSet rs = ps.executeQuery();
> ```
> Разница не только в безопасности (защита от SQL injection через параметризацию), но и в производительности — БД может закэшировать план выполнения по тексту запроса и переиспользовать его при разных значениях параметров. Hibernate **всегда** генерирует `PreparedStatement` под капотом, даже для JPQL/HQL — сырых `Statement` с конкатенацией строк в Hibernate нет.

>[!question]- Три метода выполнения `Statement`
> - `executeQuery(sql)` — для `SELECT`, возвращает `ResultSet`.
> - `executeUpdate(sql)` — для `INSERT`/`UPDATE`/`DELETE`/DDL, возвращает `int` (число затронутых строк).
> - `execute(sql)` — универсальный, когда заранее не известно, `SELECT` это или нет (например, вызов хранимой процедуры); проверяется через `getResultSet()`/`getUpdateCount()`.

## ResultSet

>[!question]- Что такое `ResultSet`
> **Курсор** по строкам, которые вернул `SELECT`-запрос. Не вся выборка сразу в памяти — `ResultSet` итерируется построчно через `next()`, а данные текущей строки читаются по имени/индексу колонки.
> ```java
> ResultSet rs = ps.executeQuery();
> while (rs.next()) {
>     Long id = rs.getLong("id");
>     String name = rs.getString("name");
> }
> ```
> `ResultSet` живёт, пока жив породивший его `Statement`/`Connection` — если закрыть `Statement` или `Connection` раньше, чем дочитан `ResultSet`, чтение упадёт с ошибкой (`ResultSet` уже закрыт). Поэтому все три объекта принято закрывать в обратном порядке создания (или через try-with-resources, что делает это автоматически).
> Hibernate именно из `ResultSet` строит Java-объекты (Entity/DTO) — это последний шаг маппинга "строка БД → объект".

>[!question]- Полный пример: сырой JDBC без Hibernate
> ```java
> try (Connection connection = dataSource.getConnection();
>      PreparedStatement ps = connection.prepareStatement("SELECT id, name FROM events WHERE id = ?")) {
>
>     ps.setInt(1, eventId);
>
>     try (ResultSet rs = ps.executeQuery()) {
>         while (rs.next()) {
>             System.out.println(rs.getLong("id") + " " + rs.getString("name"));
>         }
>     }
> } // connection.close() здесь — либо реальное закрытие, либо возврат в пул (см. DataSource)
> ```

## DataSource

>[!question]- Что такое `DataSource` — поставщик JDBC-соединений
> `javax.sql.DataSource` (Jakarta: `jakarta.sql.DataSource`) — интерфейс-фабрика для получения `Connection`. Пришёл на замену более старому `DriverManager.getConnection(url, user, password)`.
> Сам по себе интерфейс ничего не говорит о пулинге — это может быть:
> - **непулящая** реализация (каждый `getConnection()` = новое физическое соединение, как раньше делал `DriverManager`) — на практике почти не используется в проде;
> - **пулящая** реализация — держит заранее открытый пул соединений и выдаёт их из пула. Именно так работает HikariCP, Apache DBCP2, Tomcat JDBC Pool, c3p0.
> Hibernate/JPA **не создают соединения сами** — на старте им подсовывается готовый `DataSource` (через `spring.datasource.*` в Spring Boot), и дальше любой JDBC-вызов внутри Hibernate идёт через `dataSource.getConnection()`.

>[!question]- Когда физически устанавливаются соединения — при каждом CRUD-запросе?
> **Нет**, не при каждом запросе — в этом и смысл пула. Разница между двумя моделями:
>
> **Без пула (`DriverManager`, наивный подход):** каждый вызов `getConnection()` = новое TCP-соединение + TLS-хендшейк + аутентификация на БД с нуля. Это дорого (десятки–сотни мс) и именно так исторически убивали производительность приложений под нагрузкой.
>
> **С пулом (HikariCP и т.п.):** физические соединения устанавливаются **заранее, при старте пула** (eager — до `minimumIdle` штук сразу) и **лениво по требованию** при росте нагрузки (до `maximumPoolSize`), а не по одному на каждый CRUD.
> Когда Hibernate/твой код вызывает `dataSource.getConnection()` на CRUD-запрос:
> - если в пуле есть **свободное** (idle) соединение — оно выдаётся мгновенно (никакой сетевой активности, просто пометка "занято", микросекунды);
> - если свободных нет, а лимит `maximumPoolSize` не достигнут — HikariCP **в этот момент** физически открывает новое соединение;
> - если лимит достигнут — запрос ждёт в очереди до `connectionTimeout`, либо падает с ошибкой (пул исчерпан).
>
> В Spring `@Transactional`-методе физическое соединение "прикрепляется" к транзакции при **первом** обращении к БД внутри неё и удерживается до `commit`/`rollback` — все запросы в рамках одной транзакции идут через одно и то же соединение.

>[!question]- Когда соединение отключается / обрывается
> Тут важно различать **"close() из кода"** и **"реальный разрыв TCP"** — с пулом это почти всегда разные события:
>
> **"Закрытие" на уровне приложения (`connection.close()` / конец `@Transactional`):**
> С пулом `close()` **не рвёт** физическое соединение — он просто **возвращает его в пул** как idle/свободное, готовое к переиспользованию следующим запросом. Это и есть весь смысл пулинга.
>
> **Реальный физический разрыв соединения происходит:**
> - когда простаивающее (idle) соединение превысило `idleTimeout` и в пуле уже есть `minimumIdle` штук — HikariCP закрывает лишнее;
> - когда соединение прожило дольше `maxLifetime` — принудительно закрывается и пересоздаётся **проактивно**, чтобы не держать соединения, которые может незаметно прибить БД/firewall/облачный LB по их собственному таймауту;
> - когда соединение **не проходит validation-проверку** при выдаче из пула (например, сеть моргнула) — HikariCP выкидывает битое соединение и открывает новое;
> - когда БД или сетевая инфраструктура сама рвёт простаивающее соединение (MySQL `wait_timeout`, таймаут облачного балансировщика) — пул узнаёт об этом только при следующей попытке использовать это соединение и заменяет его;
> - при остановке приложения — `HikariDataSource.close()` рвёт все соединения пула разом (это именно та строка `HikariPool-1 - Shutdown completed` в логах при остановке контекста Spring).
>
> Итого: обычный CRUD-запрос **не открывает и не закрывает** физическое соединение — он берёт готовое из пула и возвращает его обратно. Реальное открытие/закрытие TCP — событие уровня пула, случается на порядок реже, чем количество запросов.