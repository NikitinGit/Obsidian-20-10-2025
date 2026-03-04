# MYSQL
1. [ ] Как удалить таблицу если в ней есть связи
2. [ ] Отчистить таблицу 
3. [ ] Что такое UNION ALL и его типы 
4. [x] какой алгоритм используется в индексах 

>[!question]- индексы
> ускоряют выборку, замедляют остальные операции  , используется алгоритм B‑tree 
> ```
> CREATE INDEX idx_battles_predicted_start_time ON battles(predicted_start_time);
> ```

>[!question]- Перенести данные с препродовой БД в локальную БД 
>создаеам дамп и скачиваем 
>```
>mariadb-dump -h 188.225.76.97 -u dev_user -p'thah2Eolcet6gouJ' \
> --skip-ssl \
>--no-tablespaces \
>strikerstat_preprod > /tmp/strikerstat_preprod_dump.sql
>```
>если хотим видеть процесс (не проверял - надо смотреть)
>```
>mariadb-dump -h 188.225.76.97 -u dev_user -p'thah2Eolcet6gouJ' \
  --skip-ssl --no-tablespaces --verbose \
  strikerstat_preprod > /tmp/strikerstat_preprod_dump.sql
>```
>Исправление кодирвки
>```
>sed 's/utf8mb4_0900_ai_ci/utf8mb4_general_ci/g; s/utf8mb3/utf8/g' /tmp/strikerstat_preprod_dump.sql > /tmp/strikerstat_preprod_dump_fixed.sql
>```
>
>Пересоздание БД 
>```
>docker exec mysql_db mysql -uroot -p'gtngtngtnN5' -e "DROP DATABASE IF EXISTS co34818_sign; CREATE DATABASE co34818_sign;"
>```
> Импортируем ИСПРАВЛЕННЫЙ дамп 
> ```
> cat /tmp/strikerstat_preprod_dump_fixed.sql | docker exec -i mysql_db mysql -uroot -p'gtngtngtnN5' co34818_sign
> ``` 
>   Проверяем количество таблиц (не обязательно)
>   ```
>    docker exec mysql_db mysql -uroot -p'gtngtngtnN5' co34818_sign -e "SHOW TABLES;" | wc -l
>   ```
>  Удаление временных дампов 
>    ```
>    rm -f /tmp/strikerstat_preprod_dump.sql /tmp/strikerstat_preprod_dump_fixed.sql && echo "Временные файлы удалены"
>    ```

>[!question]- Создать дамп БД strikerstat_test
> mysqldump -h 188.225.76.97 -u dev_user -p'thah2Eolcet6gouJ' strikerstat_test > /tmp/strikerstat_test_dump.sql 
> или
>  mysqldump -h 188.225.76.97 -u dev_user -p'thah2Eolcet6gouJ' --skip-ssl strikerstat_test > /tmp/strikerstat_test_dump.sql

>[!question]- Создать дамп БД strikerstat_preprod
> mysqldump -h 188.225.76.97 -u dev_user -p'thah2Eolcet6gouJ' strikerstat_preprod> /tmp/strikerstat_preprod_dump.sql 
>  mysqldump -h 188.225.76.97 -u dev_user -p'thah2Eolcet6gouJ' --skip-ssl strikerstat_preprod > /tmp/strikerstat_preprod_dump.sql

>[!question]-  Переименовать таблицу 
> RENAME TABLE judges_scores_new TO judges_scores; 

>[!question]-  соритровка таблицы по числу записей с одинаковым значением поля event_id
>```
>SELECT event_id, COUNT(*) as records_count
>FROM judges_round_scores GROUP BY event_id ORDER BY records_count ASC;
>```

>[!question]-  Удалить таблицу 
> DROP TABLE IF EXISTS judges_scores;

>[!question]-  Исправление collation для MySQL 5.7 
>Collation в MySQL — это **набор** правил сравнения и сортировки строк для конкретной кодировки 
>Character set определяет, какие символы можно хранить (например, utf8mb4), а collation — как эти символы сравниваются и сортируются.
>  sed 's/utf8mb4_0900_ai_ci/utf8mb4_general_ci/g; s/utf8mb3/utf8/g' /tmp/strikerstat_preprod_dump.sql > /tmp/strikerstat_preprod_dump_fixed.sql

>[!question]- подключиться по ssh  к тестингу а потом к mysql
>mysql -ureadonly_user -piek7IequEiJ2oLac localhost strikerstat_test ? 

>[!question]- подключиться к БД
>mysql -ureadonly_user -piek7IequEiJ2oLac -h188.225.76.97 strikerstat

>[!question]- Создать пользователя БД
>CREATE USER 'your_username'@'localhost' IDENTIFIED BY 'your_password';  
GRANT ALL PRIVILEGES ON service_db1.* TO 'your_username'@'localhost';  
FLUSH PRIVILEGES;

>[!question]- Создать пользователя БД с доступом с  любого хоста 
>CREATE USER 'your_username'@'%' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON service_db1.* TO 'your_username'@'%';
FLUSH PRIVILEGES;

>[!question]- Установить максимальные права пользователя БД с именем root
>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION; 
>FLUSH PRIVILEGES;

>[!question]- Посмотреть созданные  БД
>SHOW DATABASES;

>[!question]- Посмотреть созданных юзеров и их хосты
>SELECT user, host FROM mysql.user;

>[!question]- Удалить пользователя бд
>DROP USER 'your_username'@'localhost'; 

>[!question]- Число записей с данным условием 
>SELECT fighter_id, event_id, COUNT(*) as duplicate_count FROM events_bids_fighters GROUP BY fighter_id, event_id HAVING COUNT(*) > 1;

>[!question]- добавить вторичный ключ short
>при создании таблицы 
>CREATE TABLE judges ( id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT, name VARCHAR(100), organizer_id INT UNSIGNED, FOREIGN KEY (organizer_id) REFERENCES organizers(id));
>
>При добавлении поля таблицы 
>ADD FOREIGN KEY (organizer_id) REFERENCES organizers(id); 

>[!question]- добавления вторичного ключа long
>ALTER TABLE `battles` 
ADD COLUMN `weight_categories_id` INT NULL AFTER `battle_id`,
ADD INDEX `idx_battles_weight_category` (`weight_categories_id`),
ADD CONSTRAINT `fk_battles_weight_category` 
FOREIGN KEY (`weight_categories_id`) 
REFERENCES `olympic_events_weight_categories` (`weight_category_id`)
ON DELETE SET NULL 
ON UPDATE CASCADE;

>[!question]- Удалить вторичный ключ
>ALTER TABLE judges DROP FOREIGN KEY fk_judges_organizer;
>fk_judges_organizer = CONSTRAINT УКАЗАННЫЙ ПРИ СОЗДАНИИИ ВТОРИЧНОГО КЛЮЧА

>[!question]- Отличие одного ко многим от одного к одному
> Один ко многим от один к одному отличается юником ADD COLUMN `main_event_id` INT UNIQUE,

>[!question]- поменять поле с нот нал на нал 
>ALTER TABLE fighters  MODIFY sport_school_name VARCHAR(50) NULL DEFAULT NULL;

>[!question]- запрос на получение количества записей
> Повторения больше одного раза - 
> SELECT first_name, last_name, COUNT(*) as duplicates
FROM `fighters` 
WHERE `fromSortition`='Yes'
GROUP BY first_name, last_name
HAVING COUNT(*) > 1;
>
SELECT fighter_id, event_id, COUNT(*) as duplicate_count FROM events_bids_fighters GROUP BY fighter_id, event_id HAVING COUNT(*) > 1;
   
>[!question]- как найти выражение а БД
>так

>[!question]- какие параметры могут быть у COUNT(*)
>|   |   |   |
|---|---|---|
|`COUNT(*)`|Все строки в результате (включая NULL и дубликаты)|`SELECT COUNT(*) FROM fighters`|
|`COUNT(column_name)`|Только NON-NULL значения в указанном столбце|`SELECT COUNT(birth_date) FROM fighters`|
|`COUNT(DISTINCT column)`|Уникальные NON-NULL значения в столбце|`SELECT COUNT(DISTINCT club) FROM fighters`|
|`COUNT(1)` / `COUNT(42)`|Все строки (аналогично COUNT(*), но некоторые СУБД оптимизируют это лучше)|`SELECT COUNT(1) FROM battles`|

>[!question]- Как добавить поле
>ALTER TABLE `battles`  
ADD COLUMN `weight_category_id` INT NULL;

>[!question]- Как удалить поле 
>ALTER TABLE `battles` DROP COLUMN `weight_category_id`;

>[!question]- УДАЛЕНИЯ ПОЛЯ СО  вторичным ключом 
>ALTER TABLE `battles` 
DROP FOREIGN KEY `fk_battles_weight_category`,
DROP INDEX `idx_battles_weight_category`,
DROP COLUMN `weight_category_id`;

>[!question]- Как соединить 2 поля в одну строку
> SELECT CONCAT(' ' + 'TIM' + 'tom')
> `SELECT` `CONCAT_WS(``' '``,` `'Tom'``,` `'Smith'``,` `'Age:'``, 34)`

>[!QUESTION]- условия в запросах
>`SELECT` `ProductName, ProductCount,`
`CASE`
`WHEN` `ProductCount = 1`
`THEN` `'Товар заканчивается'`
`WHEN` `ProductCount = 2`
`THEN` `'Мало товара'`
`WHEN` `ProductCount = 3`
`THEN` `'Есть в наличии'`
`ELSE` `'Много товара'`
`END` `AS` `Category`
`FROM` `Products;`  
>
SELECT CASE WHEN COUNT(t) > 0 THEN true ELSE false END FROM JudgeScore t 
>
>`SELECT` `ProductName, Manufacturer,`
`IF(ProductCount > 3,` `'Много товара'``,` `'Мало товара'``)`
`FROM` `Products;`

>[!question]-  COALESCE это 
>Функция COALESCE принимает список значений и возвращает первое из них, которое не равно NULL:
> `COALESCE``(выражение_1, выражение_2, выражение_N)` |




>[!question]- Как работают индексы в MySQL? Зачем они нужны?
>## Что такое индекс?
>**Индекс** = ускорение выборки данных (SELECT с WHERE/ORDER BY/JOIN/GROUP BY)
>
>### Аналогия с книгой:
>- **Без индекса** = книга без оглавления. Чтобы найти информацию, нужно пролистать все страницы подряд (**Full Table Scan**)
>- **С индексом** = книга с оглавлением. Вы сразу видите на какой странице нужная информация и переходите туда напрямую
>
>---
>
>## Пример: поиск боев по времени
>
>### Без индекса на `predicted_start_time`:
>```sql
>SELECT * FROM battles WHERE predicted_start_time > '2026-02-03 10:00:00';
>```
>MySQL должен **просканировать ВСЕ строки** таблицы `battles` (например, 100 000 боев) и проверить каждую строку: подходит ли она по условию.
>
>### С индексом на `predicted_start_time`:
>MySQL использует индекс (отсортированную структуру данных - **B-Tree**), который хранит все значения `predicted_start_time` в отсортированном виде с указателями на строки. MySQL **сразу находит** нужный диапазон времени и извлекает только подходящие строки.
>
>---
>
>## Техническая реализация
>
>Индекс в MySQL (B-Tree) работает как отсортированное дерево:
>
>```
>                [12:00]
>               /       \
>        [10:00]         [14:00]
>       /      \         /      \
>   [09:00] [11:00] [13:00] [15:00]
>     ↓       ↓       ↓       ↓
>  строка1  строка2 строка3 строка4
>```
>
>**Сложность:**
>- Поиск в дереве = **O(log N)** (быстро)
>- Сканирование таблицы = **O(N)** (медленно)
>
>---
>
>## ✅ Когда индексы ПОЛЕЗНЫ
>
>Индексы ускоряют:
>- Частые **SELECT с WHERE** по этим полям
>- **ORDER BY** по этим полям
>- **JOIN** по этим полям
>- **GROUP BY** по этим полям
>
>**Пример:**
>```sql
>-- Быстро с индексом
>SELECT * FROM battles
>WHERE predicted_start_time BETWEEN '2026-02-03 10:00' AND '2026-02-03 12:00'
>ORDER BY predicted_start_time;
>```
>
>---
>
>## ❌ Когда индексы ВРЕДНЫ
>
>Индексы замедляют:
>- **INSERT/UPDATE/DELETE** (нужно обновлять индекс)
>- Занимают **дополнительное место на диске**
>- Если колонка **редко используется** в WHERE/ORDER BY
>
>---
>
>## Варианты создания индексов
>
>### Вариант 1: Три отдельных индекса
>```sql
>CREATE INDEX idx_battles_predicted_start_time ON battles(predicted_start_time);
>CREATE INDEX idx_battles_planned_start_time ON battles(planned_start_time);
>CREATE INDEX idx_battles_fact_start_time ON battles(fact_start_time);
>```
>
>**Хорошо для запросов:**
>```sql
>-- Каждый запрос использует свой индекс
>SELECT * FROM battles WHERE predicted_start_time > '2026-02-03 10:00';
>SELECT * FROM battles WHERE planned_start_time < '2026-02-03 12:00';
>SELECT * FROM battles WHERE fact_start_time IS NOT NULL;
>```
>
>---
>
>### Вариант 2: Составной индекс
>```sql
>CREATE INDEX idx_battles_event_times
>ON battles(event_id, predicted_start_time, planned_start_time, fact_start_time);
>```
>
>**Хорошо для запросов с комбинацией:**
>```sql
>-- Использует составной индекс эффективно
>SELECT * FROM battles
>WHERE event_id = 123
>  AND predicted_start_time > '2026-02-03 10:00'
>ORDER BY planned_start_time;
>```
>
>**⚠️ Правило левой части индекса:**
>Составной индекс работает **только если запрос использует колонки слева направо**:
>- ✅ `event_id` → работает
>- ✅ `event_id, predicted_start_time` → работает
>- ❌ `predicted_start_time` (без event_id) → **НЕ работает**
>
>---
>
>## 💡 Рекомендация
>
>**Если часто ищете бои по `event_id + время`:**
>```sql
>CREATE INDEX idx_battles_event_predicted_time
>ON battles(event_id, predicted_start_time);
>```
>
>**Если ищете бои только по времени (без event_id):**
>```sql
>CREATE INDEX idx_battles_predicted_start_time ON battles(predicted_start_time);
>```  

