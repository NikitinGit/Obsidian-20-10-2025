# MAVEN

1. [x] модули
2. [x] плагины
3. [x] жизненный цикл мавен
4. [ ] установить на впн
5. [x] ci/cd
6. [x] отличие плагинов от зависимостей в мавен
7. [x] создать свою зависимость и добавить в другой проект
8. [x] создать свой плагин и добавить в другой проект

---

## Основы для собеседования

>[!question]- Что такое Maven?
> **Maven** — это инструмент для **сборки**, **управления зависимостями** и **управления проектом** на JVM (Java, Kotlin, Scala, Groovy).
> Ключевые идеи:
> - **Convention over Configuration** — стандартная структура проекта (`src/main/java`, `src/test/java`, `src/main/resources` и т.д.) и стандартные фазы сборки.
> - **Декларативное описание** проекта в одном XML-файле — `pom.xml` (Project Object Model).
> - **Централизованные репозитории** (Maven Central) для загрузки зависимостей по координатам `groupId:artifactId:version`.

>[!question]- На чём написан Maven?
> На Java, но может собирать проекты на других JVM-языках: Kotlin, Scala, Groovy, а через плагины — даже не-JVM (например, frontend-maven-plugin).

>[!question]- Координаты артефакта (GAV)
> Каждый артефакт в Maven идентифицируется тройкой:
> - **groupId** — обычно reverse-domain организации (`com.example`)
> - **artifactId** — имя проекта/модуля (`my-app`)
> - **version** — версия (`1.0.0-SNAPSHOT`)
>
> Полное обозначение: `com.example:my-app:1.0.0`.
> Иногда добавляют `<packaging>` (`jar`, `war`, `pom`) и `<classifier>` (например, `sources`, `javadoc`).

>[!question]- Отличие Maven от Ant и Gradle
> **Ant** — самый низкоуровневый. Сборка описывается в `build.xml`, где вручную прописываются все задачи. Управление зависимостями требует Ivy.
> **Gradle** — скриптовый DSL (Groovy/Kotlin), даёт большую гибкость и возможности программирования сборки. Использует инкрементальную сборку и кеш — обычно быстрее.
> **Maven** — основан на стандартизированной структуре проекта, декларативном `pom.xml` и централизованном управлении зависимостями. Менее гибкий, но более предсказуемый.

---

## Зависимости

>[!question]- Что такое зависимость в Maven?
> **Зависимость (dependency)** — внешняя библиотека (JAR), от которой зависит твой код. Объявляется в `<dependencies>` через координаты GAV:
> ```xml
> <dependency>
>     <groupId>org.springframework.boot</groupId>
>     <artifactId>spring-boot-starter-web</artifactId>
>     <version>3.3.2</version>
> </dependency>
> ```
> Maven скачивает её из репозитория (Maven Central, локальный `~/.m2/repository`, корпоративный Nexus/Artifactory) и добавляет в classpath на нужной фазе.

>[!question]- Что такое транзитивная зависимость?
> **Транзитивно** 1 зависимость подтягивает другую  = "по цепочке через посредника". В Maven это значит, что зависимость попала в проект **не напрямую**, а через другую зависимость.
>
> **Пример из проекта TestLinux:** в `pom.xml` явно указан только `spring-boot-starter-data-jpa`. Но `mvn dependency:tree` показывает:
> ```
> +- spring-boot-starter-data-jpa  ← прямая зависимость
> |  +- spring-boot-starter-aop    ← транзитивная (глубина 2)
> |  |  └─ aspectjweaver           ← транзитивная (глубина 3)
> ```
> `starter-aop` не указывался явно, но оказался в classpath, потому что его указал в своём `pom.xml` сам `starter-data-jpa`.
>
> Maven по умолчанию подтягивает **всё дерево** транзитивных зависимостей.

>[!question]- Прямая vs транзитивная — таблица
> | Тип | Где указана | Пример |
> |---|---|---|
> | **Прямая** (direct) | В твоём `pom.xml` явно | `spring-boot-starter-web`, `mysql-connector-j`, `lombok` |
> | **Транзитивная** (transitive) | В `pom.xml` другой зависимости | `aspectjweaver`, `tomcat-embed-core`, `jackson-databind`, `hibernate-core` |
>
> **Опасность неявности:** если AOP работает только потому, что JPA случайно притянула `starter-aop` — то при удалении JPA AOP сломается. Хорошая практика: **то, что реально используешь напрямую, объявлять явно**.

>[!question]- Как посмотреть дерево зависимостей?
> ```bash
> mvn dependency:tree                          # всё дерево
> mvn dependency:tree -Dincludes=*:aspectj*    # фильтр по имени
> mvn dependency:tree -Dverbose                # с пометками о конфликтах
> ```
> Полезные команды:
> ```bash
> mvn dependency:analyze            # неиспользуемые объявленные + используемые необъявленные
> mvn dependency:resolve            # скачать все зависимости
> mvn dependency:list               # плоский список
> ```

>[!question]- Scope зависимостей и транзитивность
> **Scope** определяет, **на каких фазах** доступна зависимость и **передаётся ли она дальше** транзитивно.
>
> | Scope | Доступна при | Включается в артефакт | Передаётся транзитивно |
> |---|---|---|---|
> | `compile` (по умолчанию) | везде | да | да |
> | `runtime` | runtime/test | да | как `runtime` |
> | `provided` | compile/test | **нет** (предоставляется средой, например сервером приложений) | **нет** |
> | `test` | только тесты | нет | **нет** |
> | `system` | как `provided`, но JAR указывается локально | нет | **нет** |
> | `import` | только в `dependencyManagement` для BOM | — | — |
>
> Например, `lombok` со scope `provided` не утянется в проект, который зависит от твоего.

>[!question]- exclusions — исключение транзитивных зависимостей
> Иногда транзитивная зависимость не нужна или конфликтует. Её можно исключить:
> ```xml
> <dependency>
>     <groupId>org.springframework.boot</groupId>
>     <artifactId>spring-boot-starter-web</artifactId>
>     <exclusions>
>         <exclusion>
>             <groupId>org.springframework.boot</groupId>
>             <artifactId>spring-boot-starter-tomcat</artifactId>
>         </exclusion>
>     </exclusions>
> </dependency>
> ```
> Типичные кейсы: убрать Tomcat и поставить Jetty; убрать старую версию SLF4J и подсунуть свою.

>[!question]- Конфликты версий и правило nearest-wins
> Если в дереве две версии одной библиотеки — Maven выбирает **ближайшую к корню** (правило *nearest definition*). При равной глубине — **первую** в порядке объявления.
>
> Принудительно зафиксировать версию можно через `<dependencyManagement>` или объявив зависимость прямо в своём `pom.xml`.
>
> Для диагностики: `mvn dependency:tree -Dverbose` помечает, какая версия выиграла и какая была отброшена (`omitted for conflict with X`).

>[!question]- ```dependencyManagement``` это
> **Специальный раздел** в `pom.xml` для **централизованного управления версиями зависимостей** (без их подключения).
> ```xml
> <dependencyManagement>
>     <dependencies>
>         <dependency>
>             <groupId>com.fasterxml.jackson.core</groupId>
>             <artifactId>jackson-databind</artifactId>
>             <version>2.17.2</version>
>         </dependency>
>     </dependencies>
> </dependencyManagement>
> ```
> Дальше в `<dependencies>` можно указывать без версии — она подставится из `dependencyManagement`:
> ```xml
> <dependency>
>     <groupId>com.fasterxml.jackson.core</groupId>
>     <artifactId>jackson-databind</artifactId>
> </dependency>
> ```
> Это особенно ценно в **многомодульных проектах** — версии задаются один раз в родителе.

>[!question]- BOM (Bill of Materials)
> **BOM** — это специальный pom-файл, содержащий **только** `<dependencyManagement>` с согласованными версиями группы артефактов. Импортируется так:
> ```xml
> <dependencyManagement>
>     <dependencies>
>         <dependency>
>             <groupId>org.springframework.cloud</groupId>
>             <artifactId>spring-cloud-dependencies</artifactId>
>             <version>2025.0.0</version>
>             <type>pom</type>
>             <scope>import</scope>
>         </dependency>
>     </dependencies>
> </dependencyManagement>
> ```
> - `<type>pom</type>` — говорит, что импортируем pom-файл, а не jar.
> - `<scope>import</scope>` — разрешает Maven "влить" все определения BOM в свой `<dependencyManagement>`.
>
> Spring Boot предоставляет BOM `spring-boot-dependencies` (его обычно подключают через `<parent>` — `spring-boot-starter-parent`).

>[!question]- properties это
> Секция в `pom.xml` для **централизованного объявления переменных**, которые можно использовать в других местах POM:
> ```xml
> <properties>
>     <spring-cloud.version>2025.0.0</spring-cloud.version>
>     <java.version>21</java.version>
> </properties>
> ```
> Использование: `<version>${spring-cloud.version}</version>`.
> Также доступны системные свойства (`${project.version}`, `${project.basedir}`, `${user.home}` и т.д.).

---

## Плагины

>[!question]- Что такое плагин и чем отличается от зависимости?
> **Зависимость (dependency)** — это **библиотека**, которая попадает в classpath приложения и используется в рантайме (или на этапе компиляции).
> **Плагин (plugin)** — это **исполняемый код для самого Maven**, который выполняет действия на фазах сборки (компиляция, упаковка, тестирование, деплой). В classpath приложения **не попадает**.
>
> | | Зависимость | Плагин |
> |---|---|---|
> | Где живёт | `<dependencies>` | `<build><plugins>` |
> | Назначение | используется кодом приложения | используется Maven для сборки |
> | В classpath | да | нет |
> | Пример | `spring-boot-starter-web` | `maven-compiler-plugin`, `spring-boot-maven-plugin` |

>[!question]- Примеры стандартных плагинов
> - **`maven-compiler-plugin`** — компилирует Java-код (можно настроить версию JDK).
> - **`maven-surefire-plugin`** — запускает unit-тесты на фазе `test`.
> - **`maven-failsafe-plugin`** — интеграционные тесты на фазе `integration-test`.
> - **`maven-jar-plugin` / `maven-war-plugin`** — упаковка артефакта.
> - **`maven-shade-plugin`** — собирает "fat jar" со всеми зависимостями.
> - **`spring-boot-maven-plugin`** — `mvn spring-boot:run`, упаковка исполняемого jar.
> - **`maven-deploy-plugin`** — публикация в удалённый репозиторий.
> - **`maven-release-plugin`** — релизный workflow (tag + bump версий).
> - **`exec-maven-plugin`** — запуск произвольных Java-классов / shell-команд.

>[!question]- Как привязать плагин к фазе (execution)
> ```xml
> <plugin>
>     <groupId>org.apache.maven.plugins</groupId>
>     <artifactId>maven-compiler-plugin</artifactId>
>     <executions>
>         <execution>
>             <id>compile</id>
>             <phase>compile</phase>
>             <goals>
>                 <goal>compile</goal>
>             </goals>
>         </execution>
>     </executions>
> </plugin>
> ```
> - **Goal** — конкретное действие плагина (например, `compile:compile`, `surefire:test`).
> - **Phase** — фаза жизненного цикла, на которой выполнится goal.
> - Большинство стандартных плагинов уже привязаны к нужным фазам автоматически.

>[!question]- Создать свой плагин
> 1. Создать Maven-проект с `<packaging>maven-plugin</packaging>`.
> 2. Добавить зависимости `maven-plugin-api` и `maven-plugin-annotations`.
> 3. Написать Mojo (Maven plain Old Java Object):
>    ```java
>    @Mojo(name = "say-hello", defaultPhase = LifecyclePhase.COMPILE)
>    public class HelloMojo extends AbstractMojo {
>        @Parameter(property = "message", defaultValue = "Hi!")
>        private String message;
>
>        public void execute() {
>            getLog().info("Plugin says: " + message);
>        }
>    }
>    ```
> 4. `mvn install` — плагин попадает в локальный репозиторий.
> 5. В другом проекте:
>    ```xml
>    <plugin>
>        <groupId>com.example</groupId>
>        <artifactId>my-hello-plugin</artifactId>
>        <version>1.0.0</version>
>    </plugin>
>    ```
> 6. Запуск: `mvn com.example:my-hello-plugin:say-hello` или через `<executions>`.

---

## Жизненный цикл Maven

>[!question]- Три встроенных lifecycle
> 1. **`clean`** — очистка артефактов прошлой сборки.
> 2. **`default` (build)** — основная сборка проекта.
> 3. **`site`** — генерация документации сайта проекта.
>
> При вызове фазы выполняются **все предыдущие фазы** того же lifecycle.

>[!question]- Основные фазы default lifecycle (по порядку)
> 1. **`validate`** — проверка корректности проекта.
> 2. **`compile`** — компиляция исходников.
> 3. **`test`** — запуск unit-тестов (не упаковывает код).
> 4. **`package`** — упаковка скомпилированного кода в jar/war.
> 5. **`verify`** — запуск интеграционных тестов и проверок качества.
> 6. **`install`** — установка артефакта в **локальный** репозиторий (`~/.m2/repository`).
> 7. **`deploy`** — публикация артефакта в **удалённый** репозиторий (Nexus / Artifactory / Central).
>
> Команды:
> ```bash
> mvn clean install              # очистить + собрать + положить в локальный repo
> mvn clean package -DskipTests  # собрать без тестов
> mvn verify                     # включает интеграционные тесты
> mvn deploy                     # деплой в удалённый репозиторий
> ```

>[!question]- Что значит `clean install`?
> Сначала выполняется `clean` lifecycle (удаляет `target/`), потом `install` lifecycle (с проходом через все фазы от `validate` до `install`). Это самый часто используемый "чистый билд".

---

## Модули и многомодульные проекты

>[!question]- Что такое модуль?
> Многомодульный (multi-module) проект — это проект из нескольких суб-проектов, объединённых общим **родительским pom-ом** с `<packaging>pom</packaging>`.
> Каждый модуль — отдельный артефакт со своим `pom.xml`.

>[!question]- Структура многомодульного проекта
> ```
> my-app/
>   pom.xml              ← parent, packaging=pom, перечислены modules
>   common/
>     pom.xml            ← модуль 1
>     src/...
>   api/
>     pom.xml            ← модуль 2
>     src/...
>   web/
>     pom.xml            ← модуль 3
>     src/...
> ```
> Родительский `pom.xml`:
> ```xml
> <packaging>pom</packaging>
> <modules>
>     <module>common</module>
>     <module>api</module>
>     <module>web</module>
> </modules>
> ```
> Модуль наследует от родителя:
> ```xml
> <parent>
>     <groupId>com.example</groupId>
>     <artifactId>my-app</artifactId>
>     <version>1.0.0</version>
> </parent>
> ```
> Зависимости и плагины можно объявлять в `<dependencyManagement>` / `<pluginManagement>` родителя — все модули их подхватят.

>[!question]- Команды для работы с модулями
> ```bash
> mvn clean install                       # собрать все модули
> mvn -pl api -am clean install           # собрать только api и зависимые модули (-am = also make)
> mvn -pl api -amd clean install          # собрать api и тех, кто от него зависит (-amd = also make dependents)
> mvn -rf api clean install               # продолжить сборку с модуля api (resume from)
> ```

---

## Репозитории и settings.xml

>[!question]- Какие бывают репозитории?
> 1. **Локальный** — `~/.m2/repository`, кеш на твоей машине.
> 2. **Центральный** — Maven Central (https://repo.maven.apache.org/maven2/), подключён по умолчанию.
> 3. **Корпоративный** — Nexus / Artifactory / GitHub Packages — для приватных артефактов.
> 4. **Зеркала (mirrors)** — настраиваются в `~/.m2/settings.xml`, могут перенаправлять запросы к Central через корпоративный прокси.
>
> Порядок поиска: локальный → удалённые (в порядке объявления).

>[!question]- settings.xml — где и зачем
> - **Глобальный**: `$M2_HOME/conf/settings.xml`
> - **Пользовательский**: `~/.m2/settings.xml`
>
> Содержит:
> - **`<servers>`** — креды для приватных репозиториев (логин/пароль/токен).
> - **`<mirrors>`** — зеркала (например, корпоративный Nexus вместо Central).
> - **`<profiles>`** — переменные окружения, переключаемые настройки.
> - **`<localRepository>`** — путь к локальному репо (по умолчанию `~/.m2/repository`).

>[!question]- Создать свою зависимость и использовать в другом проекте
> **Вариант 1 — локально:**
> 1. В проекте библиотеки: `mvn clean install` → артефакт попадёт в `~/.m2/repository`.
> 2. В другом проекте просто добавить `<dependency>` с теми же GAV.
>
> **Вариант 2 — через приватный репозиторий (Nexus/Artifactory/GitHub Packages):**
> 1. В библиотеке настроить `<distributionManagement>`:
>    ```xml
>    <distributionManagement>
>        <repository>
>            <id>my-nexus</id>
>            <url>https://nexus.example.com/repository/maven-releases/</url>
>        </repository>
>    </distributionManagement>
>    ```
> 2. Креды положить в `~/.m2/settings.xml` под тем же `<id>my-nexus</id>`.
> 3. `mvn deploy` — артефакт улетит в репо.
> 4. В другом проекте указать `<repositories>` (если не Central) и `<dependency>`.

---

## Профили

>[!question]- Что такое профиль (profile)?
> **Профиль** — набор настроек (зависимости, плагины, properties), активируемых по условию. Используются для разных окружений (dev/prod), JDK, ОС и т.д.
> ```xml
> <profiles>
>     <profile>
>         <id>prod</id>
>         <properties>
>             <env>production</env>
>         </properties>
>     </profile>
> </profiles>
> ```
> Активация:
> ```bash
> mvn clean install -Pprod                 # вручную
> ```
> Также можно активировать по условиям: `<activeByDefault>`, `<jdk>`, `<os>`, `<property>`, `<file>`.

---

## CI/CD

>[!question]- Интеграция Maven с CI/CD
> Типовой пайплайн (GitHub Actions / GitLab CI / Jenkins):
> 1. **Checkout** кода.
> 2. **Setup JDK** + кеш `~/.m2/repository` (ускоряет билды).
> 3. **`mvn -B clean verify`** — `-B` (batch mode) отключает интерактивность. Включает компиляцию + unit + integration тесты.
> 4. **Артефакты** — `target/*.jar` сохраняются как build artifacts.
> 5. **`mvn deploy`** на release-веткой/теге — публикация в Nexus/реестр.
> 6. Контейнеризация: `docker build` → `docker push`.
>
> **Полезные флаги для CI:**
> - `-B` — batch mode (без цвета и интерактивности).
> - `-T 4` — параллельная сборка модулей (4 потока).
> - `-Dmaven.test.failure.ignore=false` — падать сразу при ошибке.
> - `--no-transfer-progress` — убирает спам про загрузку зависимостей в логе.
> - `-DskipTests` vs `-Dmaven.test.skip=true` — первое **пропускает запуск** тестов, второе **не компилирует** их вообще (быстрее, но рискованнее).

>[!question]- Пример GitHub Actions для Maven
> ```yaml
> name: build
> on: [push, pull_request]
> jobs:
>   build:
>     runs-on: ubuntu-latest
>     steps:
>       - uses: actions/checkout@v4
>       - uses: actions/setup-java@v4
>         with:
>           java-version: '21'
>           distribution: 'temurin'
>           cache: maven
>       - run: mvn -B --no-transfer-progress clean verify
> ```

---

## Полезные команды и приёмы

>[!question]- Запустить отдельный класс через Maven
> ```bash
> cd /home/igor/IdeaProjects/TestLinux
> mvn exec:java -Dexec.mainClass="com.example.testlinux.blind.seal.BlindSeal"
> ```
> Чтобы обновить приложение — `mvn clean compile` (находясь в папке с `pom.xml`).

>[!question]- Запустить Spring проект
> ```bash
> mvn spring-boot:run
> ```
> На определённом порту:
> ```bash
> mvn spring-boot:run -Dspring-boot.run.arguments="--server.port=8082"
> ```

>[!question]- Обновить проект со сбросом кеша (force-update SNAPSHOT)
> ```bash
> mvn clean install -DskipTests -U
> ```
> Флаг `-U` заставляет Maven перепроверить snapshot-версии в удалённых репозиториях.

>[!question]- Узнать путь, где установлен Maven (WSL/Linux)
> ```bash
> which mvn
> mvn -v                # версия + путь к Maven Home + JAVA_HOME
> ```

>[!question]- Полезные команды (шпаргалка)
> ```bash
> mvn dependency:tree                  # дерево зависимостей
> mvn dependency:analyze               # неиспользуемые / необъявленные
> mvn help:effective-pom               # итоговый pom с учётом наследования и профилей
> mvn help:effective-settings          # итоговый settings.xml
> mvn versions:display-dependency-updates  # есть ли более новые версии
> mvn versions:display-plugin-updates  # есть ли более новые версии плагинов
> mvn site                             # генерация HTML-сайта проекта
> mvn install -o                       # offline-режим
> ```

---

## Что часто спрашивают на собесе

- В чём разница между `dependency` и `plugin`? → классификатор кода для рантайма vs код для Maven.
- Что такое транзитивные зависимости и как с ними бороться? → `exclusions`, `dependencyManagement`, BOM.
- Как Maven разрешает конфликты версий? → nearest-wins + порядок объявления.
- Что делает `mvn clean install`? → clean lifecycle + default lifecycle до фазы install (локальный repo).
- Чем `install` отличается от `deploy`? → локальный vs удалённый репозиторий.
- Что такое scope `provided`? → есть на этапе компиляции/тестов, но не упаковывается (предоставляется средой, например сервером приложений).
- Какая разница между `dependencyManagement` и `dependencies`? → первое только описывает версии, второе реально подключает.
- Что такое BOM? → pom с `dependencyManagement`-секцией, импортируется через `import` scope.
- Что такое `SNAPSHOT` версия? → нестабильная, перезаписываемая в репо; Maven перепроверяет её регулярно (по умолчанию раз в день).
- Чем `package` отличается от `install`? → первое только упаковывает в `target/`, второе ещё кладёт в `~/.m2`.