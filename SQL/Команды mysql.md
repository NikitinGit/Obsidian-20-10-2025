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

>[!question]- добавления вторичного ключа 
>ALTER TABLE `battles` 
ADD COLUMN `weight_categories_id` INT NULL AFTER `battle_id`,
ADD INDEX `idx_battles_weight_category` (`weight_categories_id`),
ADD CONSTRAINT `fk_battles_weight_category` 
FOREIGN KEY (`weight_categories_id`) 
REFERENCES `olympic_events_weight_categories` (`weight_category_id`)
ON DELETE SET NULL 
ON UPDATE CASCADE;

>[!question]- УДАЛЕНИЯ ПОЛЯ СО  вторичным ключом 
>ALTER TABLE `battles` 
DROP FOREIGN KEY `fk_battles_weight_category`,
DROP INDEX `idx_battles_weight_category`,
DROP COLUMN `weight_category_id`;

>[!question]- Как седенить 2 поля в одну строку
> SELECT CONCAT(' ' + 'TIM' + 'tom')
> `SELECT` `CONCAT_WS(``' '``,` `'Tom'``,` `'Smith'``,` `'Age:'``, 34)`

>[!QUESTION]- условия в запросых
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







