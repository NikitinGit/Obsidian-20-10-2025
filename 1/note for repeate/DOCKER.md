#flashcards

1. [ ] смотреть логи - https://www.google.com/search?udm=50&sourceid=chrome&ccb=1&cs=1&hl=en-US&q=%D0%BA%D0%B0%D0%BA+%D0%B7%D0%B0%D0%BF%D1%83%D1%81%D1%82%D0%B8%D1%82%D1%8C+docker+logs+%D1%87%D1%82%D0%BE%D0%B1%D1%8B+%D0%B2%D0%B8%D0%B4%D0%B5%D1%82%D1%8C+%D0%BB%D0%BE%D0%B3%D0%B8+%D0%B2+%D1%80%D0%B0%D0%BD%D1%82%D0%B0%D0%B9%D0%BC%D0%B5&gs_lcrp=EgZjaHJvbWUyBggAEEUYOTIHCAEQIRigATIHCAIQIRigATIHCAMQIRigATIHCAQQIRigATIHCAUQIRifBdIBCTE2NTE0ajBqN6gCALACAA&ie=UTF-8&fbs=ADc_l-bs2zYa4_vOddERTHH0rF8CDbOPXIO2Jmt3O1q1mG8sqIBodZU9EysgsK8u7QK6IWDOptN89aOQj40J5PwHNgDxoMwPOlM4YRGOS6sBFB0xNiWcJvUB9-984xVL_FDQJZMzwVuMMVdUmLm4_ytgIX8K0XjgtAjEWZmArpwSN1edSe-6XWT6DtruYUiIxSLLxS14rVxgzyc3HdOoTsnkDnzX6VEiqcb_m1m-eg7T96Y6D1cfe8VzDXcaNtyNN5MUGiSr63j-vxQ43gcbH4i3ctYDR5M88w&aep=10&ntc=1&mstk=AUtExfDFmhDstDShInzgHv7x1gt3dPpgwsYB1z9XLY4EZElujThzoiNbNmZW0jMsosKkto0J9OlwuBkybyIkUCEr8-8X44RjfNUMRTYzVfSesyPZeyQvY3lTsNgBrfm2lrHQlO5N3TmowxsBZlsHLzcEc0wTAn26CbnaqMlQ75T0vQXorAeBrNGiQxu4dSLfYTsP6O3X4RiErwTZnFEKNGRmbZsDAvAK7mOaXe-Cio2zrThLv_y0FfZTJhn7N2t1nsMKOwIifU75-1uH_g&aioh=3&csuir=1&biw=2048&bih=1056.800048828125&mtid=QDcparWFA8KlptQPoJ-F6QY 

 > [!question]- посмотреть логи в рил тайме с момента запуска команды
 > docker logs -f --since 0m <имя_контейнера>
<!--SR:!fsrs,2026-11-23T09:57:59.882Z,8,8.2956,1,2,1,0,0,2026-11-15T09:57:59.882Z,id=dif23f-->

 > [!question]- посмотреть логи содержаший искомый текст
 > docker logs 9e4 | grep YooKassaService
<!--SR:!fsrs,2026-11-27T08:34:17.620Z,10,8.2956,1,2,1,0,0,2026-11-17T08:34:17.620Z,id=jobk6a-->
 
> [!question]-  Найти последние 3 записи на искомый текст
>  **Ответ:**  docker logs 561 | grep -A 3 YooKassaService
<!--SR:!fsrs,2026-11-23T09:57:41.451Z,8,8.2956,1,2,1,0,0,2026-11-15T09:57:41.451Z,id=rwd4ed-->

>[!question]- Подключиться к контейнеру
>docker exec -it f28 bash - запускает внутри контейнера интерактивную оболочку
>bash - после чего можно запускать команды линукс, bash shell
>docker exec -it -u root f28 bash - запуск с конкретным пользователем
>docker exec -it -e VAR=value f28 bash - запуск с переменными окружения
>docker exec -it f28 sh - если нет bash
<!--SR:!fsrs,2026-11-23T09:56:24.292Z,8,8.2956,1,2,1,0,0,2026-11-15T09:56:24.292Z,id=ai10bs-->

>[!question]- подключиться к БД серверу если он запущен в докере
>Подключение к контейнеру
>docker exec -it f28 mysql -u root -p
>создание БД
>CREATE DATABASE service_db1;
<!--SR:!fsrs,2026-11-25T09:55:57.771Z,10,8.2956,1,2,1,0,0,2026-11-15T09:55:57.771Z,id=g522yv-->

>[!question]- как написать 2 докер файла java приложения, использующих один jdk
>Пиши
<!--SR:!fsrs,2026-11-23T09:56:05.028Z,8,8.2956,1,2,1,0,0,2026-11-15T09:56:05.028Z,id=wr3zy3-->

>[!question]- Типы томов: `volume`, `bind`, `tmpfs`.
>do
<!--SR:!fsrs,2026-11-23T08:34:14.052Z,6,8.2956,1,2,1,0,0,2026-11-17T08:34:14.052Z,id=vudec0-->

>[!question]-  Виды сетей: `bridge`, `host`, `overlay`.
>do it
<!--SR:!fsrs,2026-11-23T09:56:57.547Z,8,8.2956,1,2,1,0,0,2026-11-15T09:56:57.547Z,id=9w4y0z--> 

>[!question]- Что такое VOLUME в докерфайле
>Внешнее хранилище контейнера , в которое можно зайти  с хост машины (ОС)
> docker volume ls    - посомтреть все созданные тома
> docker volume inspect local-nginx-proxy_mysql_data - посмотреть пути где они находятся , дату создания и остальные параметры
> sudo ls /var/lib/docker/volumes/local-nginx-proxy_mysql_data/_data -  посомтреть содержимое
> sudo -i
>root@igor-desktop:~/ > 1234
>zsh: command not found: 1234
>root@igor-desktop:~/ > cd /var/lib/docker/volumes/local-nginx-proxy_mysql_data/_data       - перейти в папку
>Том - это постоянное хранилище  данных вне контейнера , в котором софт , расположенный в контейнере может сохранять данные . Без VOLUME данные хранятся внутри контейнера
>пример
>...
>VOLUME /var/lib/mysql
<!--SR:!fsrs,2026-11-25T09:55:45.264Z,10,8.2956,1,2,1,0,0,2026-11-15T09:55:45.264Z,id=my3zfz-->

>[!question]- Где может находится докер файл
>Пиши
<!--SR:!fsrs,2026-11-26T08:35:33.666Z,9,8.2956,1,2,1,0,0,2026-11-17T08:35:33.666Z,id=79oqtx-->

>[!question]- Docker Swarm это
>пиши
<!--SR:!fsrs,2026-11-23T09:57:44.989Z,8,8.2956,1,2,1,0,0,2026-11-15T09:57:44.989Z,id=3jwega-->

>[!question]- отличие образа от вируальной среды типа virtualBox
>докер использует ядро хостовой ОС , меньше весит изоляция тольк она уровне процессов и файловой системы а не полная изоляция со своим ядром как в виртуалбокс
<!--SR:!fsrs,2026-11-23T08:36:13.949Z,6,8.2956,1,2,1,0,0,2026-11-17T08:36:13.949Z,id=577ggy-->

>[!question]- Что делает команда FROM  в докерфайле
>СКАЧИВАЕТ образ из докер хаба
>Команда `FROM` указывает **базовый слой**, на котором будет построен твой образ.
<!--SR:!fsrs,2026-11-25T09:58:02.944Z,10,8.2956,1,2,1,0,0,2026-11-15T09:58:02.944Z,id=oihhr8-->

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
<!--SR:!fsrs,2026-11-23T09:57:11.590Z,8,8.2956,1,2,1,0,0,2026-11-15T09:57:11.590Z,id=wrxpvj-->

>[!question]- запустить образ
>-d - in detach mode
>docker run -d -p 8080:8080 --name my-service1 image-name
<!--SR:!fsrs,2026-11-24T09:55:21.965Z,9,8.2956,1,2,1,0,0,2026-11-15T09:55:21.965Z,id=sb0kx4-->

>[!question]- перезапустить образ
>docker stop id
>docker rm id
>docker run -d -p 8081:8080 --name my-service1 service1
<!--SR:!fsrs,2026-11-25T09:57:18.629Z,10,8.2956,1,2,1,0,0,2026-11-15T09:57:18.629Z,id=zuhxxj-->

>[!question]-  Что означает запись 8082:8080
>8082 - порт на котором слушает контейнер
>8000 - порт на который контейнер перенаправляет полученный запрос
<!--SR:!fsrs,2026-11-27T08:34:06.397Z,10,8.2956,1,2,1,0,0,2026-11-17T08:34:06.397Z,id=40bcxe-->

>[!question]- ENTRYPOINT это
>инструкция по запуску контейнера
<!--SR:!fsrs,2026-11-25T08:34:09.671Z,8,8.2956,1,2,1,0,0,2026-11-17T08:34:09.671Z,id=0phul9--> 

>[!question]- смотреть логи в рилтайме
>docker logs -f 1f1
<!--SR:!fsrs,2026-11-24T09:55:52.195Z,9,8.2956,1,2,1,0,0,2026-11-15T09:55:52.195Z,id=m06w9g--> 

>[!question]- Удаление контейнера
>Перед удалением надо остановить контейнер docker stop idContainer
>docker rm idContainer
>Принудительно удалить колнтейнер
>docker rm -f idContainer   ----  (-f = forse)
>Проверить что контейнер удален
>docker ps -a | grep nginx-proxy
<!--SR:!fsrs,2026-11-22T09:57:50.617Z,7,8.2956,1,2,1,0,0,2026-11-15T09:57:50.617Z,id=p3h1op-->  

>[!question]- Удалить образы
>```
>for image_id in $IMAGE_IDS; do     echo "Удаление контейнеров для образа: $image_id";     docker rm -f $(docker ps -a -q --filter "ancestor=$image_id") 2>/dev/null || true; done
>docker rmi -f e7af48eb40e0 594b307bda62 21333ca8662c
>```
<!--SR:!fsrs,2026-11-23T09:56:18.860Z,8,8.2956,1,2,1,0,0,2026-11-15T09:56:18.860Z,id=pgzelk-->

# ФЛАГИ
В разных командах флаги имеют разное значение 
>[!question]- docker exec -it
>Выполнить команду внутри контейнера
>**(--tty)** - выделяет псевдо-терминал для сессии
<!--SR:!fsrs,2026-11-23T08:35:58.926Z,6,8.2956,1,2,1,0,0,2026-11-17T08:35:58.926Z,id=qnil3o-->

>[!question]- docker build -t service1 .
> -t устанавливает имя образу
> . указывает на текущую директорию, где находится Dockerfile и файлы для сборки
<!--SR:!fsrs,2026-11-25T08:33:57.440Z,8,8.2956,1,2,1,0,0,2026-11-17T08:33:57.440Z,id=ejokvm-->

>[!question]- -d
>запуск контейнера в фоновом режиме detached mode
<!--SR:!fsrs,2026-11-22T09:57:15.566Z,7,8.2956,1,2,1,0,0,2026-11-15T09:57:15.566Z,id=03a2p6-->

>[!question]- -i
>interactive mode  - Подключает STDIN (стандартный ввод)
<!--SR:!fsrs,2026-11-25T08:34:41.822Z,8,8.2956,1,2,1,0,0,2026-11-17T08:34:41.822Z,id=k8f13k-->


> [!question]- Чем отличаются Dockerfile, образ и контейнер?
> **Dockerfile** — инструкция по сборке образа. **Образ (image)** — готовый набор программ и зависимостей. **Контейнер** — запущенный экземпляр образа.
>
> `Dockerfile → docker build → образ → docker run → контейнер`.
> В команде `docker run --rm mysql:8.0 mysqldump ...` используется готовый образ `mysql:8.0`. Docker скачает его, если образа нет локально, и запустит внутри контейнера утилиту `mysqldump`.
<!--SR:!fsrs,2026-11-23T09:56:51.049Z,8,8.2956,1,2,1,0,0,2026-11-15T09:56:51.049Z,id=km7ahk-->

> [!question]- Чем отличаются docker run и docker exec?
> `docker run [параметры Docker] ОБРАЗ КОМАНДА [её параметры]` — создаёт новый контейнер и запускает в нём команду.
> `docker exec [параметры Docker] КОНТЕЙНЕР КОМАНДА [её параметры]` — запускает дополнительную команду в уже работающем контейнере.
> Для разового экспорта удобно использовать `run`. Если подходящий контейнер уже работает и содержит `mysqldump`, можно использовать `exec`. В обоих случаях утилита может подключаться к удалённой БД по адресу из `-h`.
<!--SR:!fsrs,2026-11-25T08:34:23.787Z,8,8.2956,1,2,1,0,0,2026-11-17T08:34:23.787Z,id=flw5gb-->

> [!question]- --rm — это команда Linux или флаг Docker? Нужен ли он для docker exec?
> `--rm` — необязательный флаг `docker run`: Docker автоматически удалит контейнер после завершения его работы. Без флага остановленный контейнер останется в списке `docker ps -a`. Образ сохраняется для следующих запусков.
> В Linux `rm` — отдельная команда удаления файлов. У `docker exec` флага `--rm` нет: после завершения дополнительной команды существующий контейнер продолжает работать.
<!--SR:!fsrs,2026-11-23T09:57:32.303Z,8,8.2956,1,2,1,0,0,2026-11-15T09:57:32.303Z,id=8q7pwy-->

> [!question]- Можно ли выгрузить удалённую БД через docker exec 28f mysqldump?
> Да. Контейнер должен работать, содержать `mysqldump` и иметь сетевой доступ к серверу БД. Сначала посмотреть контейнеры и проверить наличие утилиты:
>
> ```bash
> docker ps
> docker exec 28f mysqldump --version
> ```
>
> `28f` — пример сокращённого ID: он должен однозначно определять нужный контейнер. Можно указать полный ID или имя из столбца `NAMES`. Затем подставить пароль БД вместо `ПАРОЛЬ` и выполнить:
>
> ```bash
> docker exec 28f mysqldump \
>   -h 188.225.76.97 -u dev_user -p'ПАРОЛЬ' \
>   --ssl-mode=DISABLED --no-tablespaces --verbose \
>   strikerstat_preprod > /tmp/strikerstat_preprod_dump.sql
> ```
>
> Завершится процесс `mysqldump`; существующий контейнер продолжит работать.
<!--SR:!fsrs,2026-11-22T09:56:33.478Z,7,8.2956,1,2,1,0,0,2026-11-15T09:56:33.478Z,id=2hezrm-->

> [!question]- Где сохраняется дамп при > /tmp/strikerstat_preprod_dump.sql и удалит ли его --rm?
> Перенаправление `>` обрабатывает оболочка, в которой введена команда. Файл создаётся на этой машине, вне контейнера, и сохраняется после удаления контейнера через `--rm`.
> `>` создаёт файл или перезаписывает уже существующий. Наличие файла само по себе не подтверждает успешную выгрузку: при ошибке он может быть пустым или неполным.
<!--SR:!fsrs,2026-11-27T08:35:50.622Z,10,8.2956,1,2,1,0,0,2026-11-17T08:35:50.622Z,id=72p44f-->

> [!question]- Зачем нужен --ssl-mode=DISABLED и обязателен ли он?
> Это параметр `mysqldump`, отключающий TLS-шифрование соединения с MySQL. Содержимое дампа передаётся по сети без шифрования на уровне MySQL.
> Флаг необязательный. По умолчанию MySQL 8.0 использует `PREFERRED`: пытается установить зашифрованное соединение, но допускает подключение без шифрования. `REQUIRED` требует шифрования и завершает подключение ошибкой, если оно невозможно.
> `DISABLED` иногда используют для обхода проблем с TLS; причину его добавления в конкретную команду нужно выяснять отдельно. Для передачи дампа через интернет лучше использовать настроенное TLS-соединение.
<!--SR:!fsrs,2026-11-23T09:57:55.561Z,8,8.2956,1,2,1,0,0,2026-11-15T09:57:55.561Z,id=e43eta-->

Источники: [Dockerfile и сборка образа](https://docs.docker.com/build/concepts/dockerfile/), [docker run и --rm](https://docs.docker.com/reference/cli/docker/container/run/#clean-up---rm), [docker exec](https://docs.docker.com/reference/cli/docker/container/exec/), [ID и имя контейнера](https://docs.docker.com/engine/containers/run/#container-identification), [режимы SSL/TLS в MySQL 8.0](https://dev.mysql.com/doc/refman/8.0/en/connection-options.html#option_general_ssl-mode).
