
# applicatioin.properties 

1. [x] как заменить на локальный 
2. [ ] контейнер сервлетов - на что влияет число подключений, запросов ?

>[!question]- как использовать локальные настройки вместо application.properties
>В IntelliJ IDEA: Run/Debug Configurations → Active profiles: local     
>или в переменных окружения узказать 
>  1. Верхняя панель → кликнуть на название конфигурации (рядом с кнопками Run/Debug)
>  2. Edit Configurations...  
>  3. Выбрать вашу конфигурацию (Spring Boot приложение) 
>  4. Вкладка Environment (или в старых версиях это поле прямо на главной вкладке)  
>  5. Поле Environment variables → вписать SPRING_PROFILES_ACTIVE=local 

