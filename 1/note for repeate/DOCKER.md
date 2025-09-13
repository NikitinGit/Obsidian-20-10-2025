
>[!question]- Подключиться к контейнеру
>docker exec -it f28 bash - запускает внутри контейнера интерактивную оболочку
>bash - после чего можно запускать команды линукс, bash shell
>docker exec -it -u root f28 bash - запуск с конкретным пользователем
>docker exec -it -e VAR=value f28 bash - запуск с переменными окружения
>docker exec -it f28 sh - если нет bash

>[!question]- подключиться к БД серверу если он запущен в докере
>Подключение к контейнеру
>docker exec -it f28 mysql -u root -p
>создание БД
>CREATE DATABASE service_db1;

>[!question]- как написать 2 докер файла java приложения, использующих один jdk
>Пиши

>[!question]- Что такое VOLUME в докерфайле
>Создает и монтирует том внутри контейнера
>Том - это постоянное хранилище  данных вне контейнера , в котором софт , расположенный в контейнере может сохранять данные . Без VOLUME данные хранятся внутри контейнера
>пример 
>...
>VOLUME /var/lib/mysql 

>[!question]- Где может находится докер файл
>Пиши

>[!question]- Docker Swarm это 
>пиши

>[!question]- отличие образа от вируальной среды типа virtualBox
>докер использует ядро хостовой ОС , меньше весит изоляция тольк она уровне процессов и файловой системы а не полная изоляция со своим ядром как в виртуалбокс

>[!question]- Что делает команда FROM  в докерфайле
>СКАЧИВАЕТ образ из докер хаба
>Команда `FROM` указывает **базовый слой**, на котором будет построен твой образ.

>[!question]- Создать образ - самый простой пример
>1. создать докер файл 
>```  
>FROM openjdk:17-jdk-slim  
>VOLUME /tmp  
>COPY target/service2-0.0.1-SNAPSHOT.jar service2-0.0.1-SNAPSHOT.jar  
>ENTRYPOINT ["java","-jar","/service2-0.0.1-SNAPSHOT.jar"]  
>```
>2. Создать образ
>docker build -t service1 .

>[!question]- запустить образ
>-d - in detach mode
>docker run -d -p 8080:8080 --name my-service1 image-name

>[!question]- перезапустить образ
>docker stop id
>docker rm id
>docker run -d -p 8081:8080 --name my-service1 service1

>[!question]-  Что означает запись 8082:8080
>8082 - порт на котором слушает контейнер
>8000 - порт на который контейнер перенаправляет полученный запрос

>[!question]- ENTRYPOINT это
>инструкция по запуску контейнера 

>[!question]- смотреть логи в рилтайме
>docker logs -f 1f1 

>[!question]- Удаление контейнера
>Перед удалением надо остановить контейнер docker stop idContainer
>docker rm idContainer
>Принудительно удалить колнтейнер
>docker rm -f idContainer   ----  (-f = forse)
>Проверить что контейнер удален
>docker ps -a | grep nginx-proxy  

>[!question]- Удалить образы
>```
>for image_id in $IMAGE_IDS; do     echo "Удаление контейнеров для образа: $image_id";     docker rm -f $(docker ps -a -q --filter "ancestor=$image_id") 2>/dev/null || true; done
>docker rmi -f e7af48eb40e0 594b307bda62 21333ca8662c
>```

# ФЛАГИ
В разных командах флаги имеют разное значение 
>[!question]- docker exec -it
>Выполнить команду внутри контейнера
>**(--tty)** - выделяет псевдо-терминал для сессии

>[!question]- docker build -t service1 .
> -t устанавливает имя образу
> . указывает на текущую директорию, где находится Dockerfile и файлы для сборки

>[!question]- -d
>запуск контейнера в фоновом режиме detached mode

>[!question]- -i
>interactive mode  - Подключает STDIN (стандартный ввод)
