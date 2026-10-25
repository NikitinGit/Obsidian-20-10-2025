
>[!question]- Копирование файлов с компа  в директорию образа
>в корень COPY pom.xml .
 в директорию COPY src ./src

>[!question]- Устанавливаем рабочую директорию
WORKDIR /app

>[!question]- Зачем нужен Dockerfile
>Dockerfile — инструкция, по которой Docker собирает **образ**: упакованное приложение + Java, которое одинаково запускается где угодно. И docker compose, и Kubernetes запускают не код и не jar, а **образы** — без Dockerfile (или его заменителя) сервис туда не попадёт.
>Сборка: `docker build -t fightlab/bid-service:latest -f bid-service/Dockerfile .` (`-t` — имя образа, `-f` — путь к Dockerfile, `.` — контекст: какие файлы доступны для `COPY`).

>[!question]- Многоэтапная сборка (multi-stage) для Spring Boot — пример с разбором
>```dockerfile
># Этап 1 — сборка: временный контейнер с Maven и JDK
>FROM maven:3.9-eclipse-temurin-21 AS build
>WORKDIR /workspace
>COPY pom.xml .
>COPY bid-service/pom.xml bid-service/
>RUN --mount=type=cache,target=/root/.m2 mvn -B -q -pl bid-service -am dependency:go-offline
>COPY bid-service/src bid-service/src
>RUN --mount=type=cache,target=/root/.m2 mvn -B -q -pl bid-service -am package -DskipTests
>RUN java -Djarmode=tools -jar bid-service/target/bid-service.jar extract --layers --launcher --destination /extracted
>
># Этап 2 — итоговый образ: только JRE + приложение
>FROM eclipse-temurin:21-jre-alpine
>RUN addgroup -S app && adduser -S app -G app
>USER app
>WORKDIR /app
>COPY --from=build /extracted/dependencies/ ./
>COPY --from=build /extracted/spring-boot-loader/ ./
>COPY --from=build /extracted/snapshot-dependencies/ ./
>COPY --from=build /extracted/application/ ./
>EXPOSE 8081
>ENTRYPOINT ["java", "-XX:MaxRAMPercentage=75", "org.springframework.boot.loader.launch.JarLauncher"]
>```
>| Приём | Зачем |
>|---|---|
>| **Два этапа** (`AS build` + `COPY --from=build`) | в итоговый образ не попадают Maven, JDK и исходники → меньше и безопаснее. Собрать можно на любой машине только с Docker, даже без Java |
>| **Сначала `pom.xml`, потом код** | Docker кэширует каждый шаг: пока зависимости не менялись, их скачивание берётся из кэша → пересборка после правки кода — секунды |
>| `--mount=type=cache,target=/root/.m2` | кэш Maven между сборками (BuildKit), зависимости не качаются заново |
>| **Jar по слоям** (`-Djarmode=tools ... extract --layers`) | зависимости (~60 МБ) меняются редко, код — часто; при обновлении образа скачивается только изменившийся слой |
>| **`USER app`** | приложение не под root: при взломе меньше прав внутри контейнера |
>| **`-XX:MaxRAMPercentage=75`** | JVM считает память от лимита **контейнера**, а не всей машины; без этого в k8s под могут убить за превышение лимита памяти (OOMKilled) |
>| `ENTRYPOINT [...]` (JSON-форма) | процесс Java — PID 1, корректно получает сигнал остановки → graceful shutdown |

>[!question]- Свой Dockerfile на каждый микросервис?
>Да, обычно у каждого сервиса **свой Dockerfile в его папке** (`bid-service/Dockerfile`, `judging-service/Dockerfile`), и каждый сервис — **отдельная запись в `services:`** docker-compose:
>```yaml
>services:
>  bid-service:
>    image: fightlab/bid-service:latest
>    build:
>      context: .                         # корень: модулю нужен родительский pom.xml
>      dockerfile: bid-service/Dockerfile
>  judging-service:
>    image: fightlab/judging-service:latest
>    build:
>      context: .
>      dockerfile: judging-service/Dockerfile
>```
>Почему отдельно: сервисы могут отличаться (версия Java, порт, системные пакеты), собираются и выкатываются независимо.
>**Нюансы:**
>- Если все сервисы собираются одинаково, можно **один общий Dockerfile с параметром**: `ARG MODULE` в Dockerfile + `build.args: { MODULE: bid-service }` в compose — меньше дублирования.
>- Секция `build:` в compose — для **локальной разработки**. В проде/k8s образ заранее собирает CI и кладёт в registry (Docker Hub, GitLab Registry), а compose/k8s только указывают `image:` и скачивают готовый.
>- Если в monorepo модулю нужен родительский `pom.xml`, `context` — корень репозитория, а не папка сервиса.

>[!question]- Как собрать образ без Dockerfile (вопрос на собесе)
>- **`./mvnw spring-boot:build-image`** — Spring Boot через Cloud Native Buildpacks, тоже по слоям, без Dockerfile.
>- **Jib** (плагин Google для Maven/Gradle) — собирает образ прямо из сборки, даже без установленного Docker.
>На работе их часто берут, чтобы не поддерживать Dockerfile руками; свой Dockerfile даёт полный контроль и понимание каждого шага.
