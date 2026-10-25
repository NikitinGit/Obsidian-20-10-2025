1. [ ] https://blog.skillfactory.ru/sql-ddl-dml-dql-dcl-tcl/ группы операторов  DDL, DML, DCL и TCL
# Группы операторов

# Сложные запросы

>[!question]- Как сделать аналог for (вставить N сгенерированных строк одним запросом)
> Цикла в обычном SQL нет — вместо него **генерируют набор строк** с номерами и вставляют их одним `INSERT ... SELECT`.
>
> **PostgreSQL — `generate_series`** (основной способ):
> ```sql
> SELECT i FROM generate_series(1, 80) AS i;   -- 80 строк: i = 1, 2, ..., 80
> ```
> **MySQL 8 / любая БД — рекурсивный CTE** (в MySQL нет `generate_series`):
> ```sql
> WITH RECURSIVE seq AS (
>     SELECT 1 AS i
>     UNION ALL
>     SELECT i + 1 FROM seq WHERE i < 80
> )
> SELECT i FROM seq;
> ```
> **Настоящий цикл — PL/pgSQL** (только когда в цикле сложная логика с условиями; иначе медленнее — N отдельных INSERT):
> ```sql
> DO $$
> BEGIN
>     FOR i IN 1..80 LOOP
>         INSERT INTO bids (event_id, fighter_id, ...) VALUES ((i - 1) / 40 + 1, (i - 1) % 40 + 1, ...);
>     END LOOP;
> END $$;
> ```
>
> **Пример: 80 заявок, 2 соревнования по 40 участников, остальное случайно** (проект microservices-kafka-k8s-lab):
> ```sql
> TRUNCATE bids RESTART IDENTITY;   -- очистить таблицу и сбросить счётчик id (новые id снова с 1)
>
> WITH generated AS (
>     SELECT i,
>            (i - 1) / 40 + 1                                                     AS event_id,   -- 1..40 → 1, 41..80 → 2
>            (i - 1) % 40 + 1                                                     AS fighter_id, -- 1..40 в каждом соревновании
>            (ARRAY ['51', '57', '63.5', '71', '80', '92'])[1 + floor(random() * 6)::int] AS weight_category,
>            (ARRAY ['PENDING', 'APPROVED', 'REJECTED'])[1 + floor(random() * 3)::int] AS status,
>            now() - random() * interval '30 days'                                AS created_at
>     FROM generate_series(1, 80) AS i
> )
> INSERT INTO bids (event_id, fighter_id, weight_category, status, created_at, updated_at)
> SELECT event_id, fighter_id, weight_category, status, created_at,
>        CASE WHEN status = 'PENDING' THEN created_at ELSE created_at + random() * (now() - created_at) END
> FROM generated;
> ```
>
> **Приёмы из примера:**
>
> | Выражение | Что даёт |
> |---|---|
> | `(i - 1) / 40 + 1` | целочисленное деление: строки 1–40 → 1, 41–80 → 2 («номер блока») |
> | `(i - 1) % 40 + 1` | остаток от деления: внутри каждого блока снова 1..40 («повтор каждые N строк») |
> | `(ARRAY[...])[1 + floor(random() * N)::int]` | случайный элемент массива из N штук; индексы массивов в PostgreSQL **с 1** |
> | `now() - random() * interval '30 days'` | случайный момент за последние 30 дней |
> | `created_at + random() * (now() - created_at)` | случайный момент между `created_at` и сейчас (`now() - created_at` — интервал, `random()` — доля от 0 до 1) |
>
> **Зачем `WITH generated AS (...)` (CTE):** случайное `created_at` сначала вычисляется один раз для строки, а потом используется для `updated_at`. Если написать `random()` в обоих местах напрямую — это будут два разных случайных числа, и `updated_at` может оказаться раньше `created_at`.
>
> **Проверить до вставки:** выделить только `WITH ... SELECT ... FROM generated` (без строки `INSERT INTO`) и выполнить — покажет строки, которые вставятся.

>[!question]- INSERT ... VALUES vs INSERT ... SELECT
> **`VALUES`** — строки пишутся руками, сколько скобок — столько строк:
> ```sql
> INSERT INTO bids (event_id, fighter_id, weight_category, status, created_at, updated_at)
> VALUES (1, 1, '71', 'PENDING', now(), now()),
>        (1, 2, '63.5', 'APPROVED', now(), now());
> ```
> **`SELECT`** — сначала выполняется `SELECT`, и **каждая строка его результата становится новой строкой** таблицы. `VALUES` не пишется. Используется, когда данные **вычисляются** (генерация, как выше) или **копируются из другой таблицы**:
> ```sql
> INSERT INTO archived_bids (id, status) SELECT id, status FROM bids WHERE created_at < '2026-01-01';
> ```
> ⚠️ Колонки `SELECT` сопоставляются с колонками в `INSERT INTO (...)` **по порядку, а не по именам**. Перепутал порядок — данные молча лягут в чужие колонки (если типы совместимы).

>[!question]- Вставка с уникальным ограничением: TRUNCATE или ON CONFLICT
> Если в таблице уникальный индекс (например, одна заявка спортсмена на соревнование — `uq_bids_event_fighter (event_id, fighter_id)`), повторяющаяся пара **уронит весь INSERT** целиком.
>
> | | `TRUNCATE` + `INSERT` | `INSERT ... ON CONFLICT DO NOTHING` |
> |---|---|---|
> | Старые данные | удаляются | остаются |
> | Конфликтующие строки | конфликтов нет | молча пропускаются |
> | `id` | с `RESTART IDENTITY` — снова с 1 | продолжают счёт |
> | Повторный запуск | снова свежие данные | вставит 0 (всё уже есть) |
>
> ```sql
> INSERT INTO bids (...) SELECT ... FROM generated
> ON CONFLICT (event_id, fighter_id) DO NOTHING;            -- колонки = колонкам уникального ограничения
> -- или по имени ограничения:
> ON CONFLICT ON CONSTRAINT uq_bids_event_fighter DO NOTHING;
> ```
> Вместо `DO NOTHING` можно `DO UPDATE SET status = EXCLUDED.status` — обновить существующую строку (upsert); `EXCLUDED` — строка, которую пытались вставить.
> В MySQL аналоги: `INSERT IGNORE` и `INSERT ... ON DUPLICATE KEY UPDATE`.

>[!question]- Приведение типа: `::int` и `CAST`
> `::` — оператор **приведения типа** (превратить значение одного типа в другой). Это **особенность PostgreSQL**, в стандарте SQL — `CAST`.
>
> | Запись | Где работает |
> |---|---|
> | `CAST(x AS int)` | стандарт SQL: PostgreSQL, MySQL (со своими названиями типов), Oracle, SQL Server… |
> | `x::int` | только PostgreSQL (и совместимые с ним БД) — короткая запись |
>
> ```sql
> SELECT CAST('42' AS int);       -- 42
> SELECT '42'::int;               -- 42, то же самое
> SELECT '2026-10-24'::date;      -- строка → дата
> SELECT now()::date;             -- время → только дата
> ```
> **Пример, где без него ошибка** — случайный элемент массива:
> ```sql
> (ARRAY ['PENDING', 'APPROVED', 'REJECTED'])[1 + floor(random() * 3)::int]
> ```
> `random()` → дробное `0.734`; `* 3` → `2.202`; `floor(...)` округляет вниз, но **тип остаётся дробным** (`2.0`, `double precision`); `::int` → целое `2`. Индекс массива обязан быть целым — без приведения PostgreSQL выдаст ошибку.
>
> **MySQL:** `::` не работает; пишется `CAST(x AS SIGNED)` / `CAST(x AS UNSIGNED)` — типа `int` в `CAST` у MySQL нет.
>
> ⚠️ **Ловушка в Spring Data:** в нативных запросах (`@Query(nativeQuery = true)`) двоеточие — это именованный параметр (`:eventId`), поэтому `::int` может быть принят за параметр `int` и сломать запрос. В Java-коде надёжнее `CAST(x AS int)` или экранирование `\\:\\:int`. В JPQL (`@Query` без `nativeQuery`) `::` нет вообще — только `CAST(x AS Integer)`.
>[!question]- count(*) vs count(колонка) vs count(DISTINCT) — почему часто одно и то же число
> | Запись | Что считает |
> |---|---|
> | `count(*)` | **строки** — неважно, что в них |
> | `count(колонка)` | строки, где в этой колонке **не `NULL`** |
> | `count(1)`, `count('x')` | то же, что `count(*)`: константа никогда не `NULL` (миф, что `count(1)` быстрее, — в PostgreSQL `count(*)` даже чуть быстрее) |
> | `count(DISTINCT колонка)` | **разные** значения (без `NULL`) |
>
> Если все колонки `NOT NULL` (как в `bids`), то `count(*)`, `count(1)`, `count(id)`, `count(status)` дают одно число. Разница появляется только при **`NULL`** или **`DISTINCT`**.
>
> **1. DISTINCT — сколько разных значений:**
> ```sql
> SELECT event_id,
>        count(*)                        AS bids,        -- 40
>        count(DISTINCT weight_category) AS categories   -- сколько разных весовых категорий
> FROM bids GROUP BY event_id;
> ```
> **2. LEFT JOIN — классическая ловушка на собесе.** Посчитать заявки у каждого соревнования, включая те, где заявок нет:
> ```sql
> SELECT e.id, count(*) AS wrong, count(b.id) AS correct
> FROM events e LEFT JOIN bids b ON b.event_id = e.id
> GROUP BY e.id;
> ```
> Для соревнования без заявок `LEFT JOIN` всё равно вернёт одну строку, где колонки `bids` = `NULL`: `count(*)` → **1** (неверно), `count(b.id)` → **0** (верно).
>
> **3. Колонка, где бывает `NULL`:** например, `count(approved_by)` — число рассмотренных заявок, `count(*)` — всех.
>
> **Подсчёт по условию — несколько счётчиков в одной строке:**
> ```sql
> SELECT event_id,
>        count(*)                                       AS total,
>        count(*) FILTER (WHERE status = 'APPROVED')    AS approved,   -- PostgreSQL
>        count(CASE WHEN status = 'PENDING' THEN 1 END) AS pending     -- везде, включая MySQL
> FROM bids GROUP BY event_id;
> ```
> Вариант с `CASE` работает именно потому, что `count(колонка)` не считает `NULL`: `CASE` без `ELSE` для неподходящих строк даёт `NULL`.
> Это «разворот в ширину»: вместо строки на каждую пару (соревнование, статус) — одна строка на соревнование, статусы колонками.

>[!question]- GROUP BY — почему из 80 строк получилось 6, и порядок выполнения запроса
> ```sql
> SELECT event_id, status, count(*) FROM bids GROUP BY event_id, status ORDER BY event_id, status;
> ```
> **Строки «схлопывает» `GROUP BY`**, а `ORDER BY` только сортирует готовый результат и количество строк не меняет.
>
> `GROUP BY event_id, status` раскладывает строки по «корзинам» — по уникальным сочетаниям значений: 2 соревнования × 3 статуса = **6 корзин** → 6 строк результата. **Агрегатные функции** (`count`, `sum`, `avg`, `min`, `max`) считаются отдельно по каждой корзине.
>
> **Порядок выполнения ≠ порядок написания:**
>
> | Шаг | Часть | Что происходит |
> |---|---|---|
> | 1 | `FROM` / `JOIN` | берём строки (здесь все 80) |
> | 2 | `WHERE` | отбрасываем ненужные **строки** |
> | 3 | **`GROUP BY`** | **строки → группы** (80 → 6) |
> | 4 | `HAVING` | отбрасываем ненужные **группы**: `HAVING count(*) > 10` |
> | 5 | `SELECT` | считаем агрегаты, выбираем колонки, назначаем псевдонимы (`AS`) |
> | 6 | `DISTINCT` | убираем дубликаты |
> | 7 | `ORDER BY` | сортируем |
> | 8 | `LIMIT` / `OFFSET` | обрезаем |
>
> **Следствия, которые спрашивают на собесе:**
> - При `GROUP BY` в `SELECT` можно писать только колонки из `GROUP BY` и агрегатные функции. `SELECT fighter_id ... GROUP BY event_id, status` → ошибка: в группе 22 строки с разными `fighter_id`, какую показать — непонятно. (Старый MySQL без режима `ONLY_FULL_GROUP_BY` молча брал случайное значение.)
> - **`WHERE` vs `HAVING`:** `WHERE` фильтрует строки **до** группировки и не видит агрегатов (`WHERE count(*) > 10` — ошибка); `HAVING` фильтрует **группы после** неё. Что можно отфильтровать в `WHERE` — фильтруй там: меньше строк попадёт в группировку.
> - Псевдоним из `SELECT` нельзя использовать в `WHERE` (он ещё не вычислен), но можно в `ORDER BY` (он выполняется после `SELECT`).
>
> ```sql
> SELECT event_id, count(*) AS approved
> FROM bids
> WHERE status = 'APPROVED'        -- строки: только одобренные
> GROUP BY event_id
> HAVING count(*) >= 15            -- группы: соревнования, где одобренных хотя бы 15
> ORDER BY approved DESC;          -- псевдоним в ORDER BY — можно
> ```
