1. ALTER TABLE `events` MODIFY COLUMN `organizer_login` INT NOT NULL;
2. ALTER TABLE `events` 
CHANGE COLUMN `organizer_login` `organizer_login` INT NOT NULL,
ADD CONSTRAINT `events_organizer_login_foreign` 
FOREIGN KEY (`organizer_login`) 
REFERENCES `users` (`id`);
3. Один ко многим от один к одному отличается юником ADD COLUMN `main_event_id` INT UNIQUE,