>[!question]- запрос на получение количества записей
> Повторения больше одного раза - 
   SELECT fighter_id, event_id, COUNT(*) as duplicate_count FROM events_bids_fighters GROUP BY fighter_id, event_id HAVING COUNT(*) > 1;

>[!question]- как найти выражение а БД
>так

[olympic_events_weight_categories 
    {
        "ring": "A",
        "fightDay": "2025-06-21",
        "timeSlot": 1,
        "fightTime": "12:00"
    },
    {
        "ring": "B",
        "fightDay": "2025-06-21",
        "timeSlot": 1,
        "fightTime": "12:00"
    },
    {
        "ring": "A",
        "fightDay": "2025-06-25",
        "timeSlot": 1,
        "fightTime": "12:00"
    }
]

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
DROP COLUMN `weight_categories_id`;



