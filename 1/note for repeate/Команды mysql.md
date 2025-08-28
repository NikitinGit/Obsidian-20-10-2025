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

>[!question]- удалить пользователя бд
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

>[!question]- Как седенить 2 поля в одну строку
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






