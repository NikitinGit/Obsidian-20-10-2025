>[!question]- Что такое volumes
>Том - пиши

>[!question]- depends_on это 
>указатель на то, что один контейнер зависит от другого, управляет порядком запуска контейнеров но не их готовностью , например 
>```
>depends_on:
>  mysql:
>    condition: service_healthy
>```
>ждет когда запуститься mysql и будет здоров - service_healthy

>[!question]- healthcheck
>```
>healthcheck:
  test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-pgtngtngtnN5"]
  interval: 5s
  timeout: 5s
  retries: 10 - после 10 попыток помечается как не здоров при docker ps
>```
>test - команда, которую докер будет выполнять пока не получит от контейнера ответа что он уже жив

>[!question]- На что указывает version: '3'
>Это не версия **Docker Compose** как утилиты, а **версия формата конфигурации** (schema version), которую использует сам Docker Compose для интерпретации файла

>[!question]- Самый простой пример микросервисного docker-compose.yml
>```yaml
>version: '3'
>services:
>  service1:
>    image: service1
>    ports:
>      - "8081:8080"
>
>  service2:
>    image: service2
>    ports:
>      - "8082:8080"
>```

>[!question]- ЗАПУСТИТЬ 
>**Cоздать в директории docker-compose.yml**
>с просмотром логов в реальном времени 
>docker-compose up 
>в фоновом режиме 
>docker-compose up -d
>Запуск с принудительной пересборкой образов
> docker-compose up -d --build
>
>**Если файл не один в проекте то ЗАПУСК ПО ИМЕНИ  **
>Запуск с принудительной пересборкой образов 
>docker-compose -f docker-compose-database.yml up --build

>[!question]- Пересобрать и запустить
>**Остановка и удаление контейнеров, сетей (но не volumes)**
>docker-compose down
>**Если нужно удалить и volumes (осторожно! данные будут потеряны)** 
  docker-compose down -v
>**Пересборка всех сервисов**
  docker-compose build

>[!question]- Удалить созданные им образы
>docker-compose down --rmi all
  
>[!question]- смотреть логи в рилтайме
>одного котейнера
>docker-compose logs -f 1f1 
>всех контейнеров
> docker-compose logs -f  

>[!question]- Указать путь к докерфайлу
> в секции servises: my-service написать 
> context: /home/user/path
> dockerfile: Dockerfile
