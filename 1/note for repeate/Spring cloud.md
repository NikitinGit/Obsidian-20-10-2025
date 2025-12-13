# Задачи 
https://chatgpt.com/c/6915e111-75d4-832a-acc4-e6004a51044c 
1. [x] Чем отличается балансировка на стороне сервера от на стороне клиента  
2. [x] Написать пример с рэббит мк 15.05 - 19.37
3. [ ] добавь реббит мк в макросервсное приложение 
4. [ ] написать пример с кафкой 
5. [ ] написать пример, где микросервисы общаются друг с другом через кафку и рэббит мк
6. [ ] hikary poll это 
7. [ ] как еврика или конфиг или что то еще может определить насколько сильно забит инстанс запросами и как сильно занят вычислениями ? что в спринг за это отвечает ?
8. [ ] напиши несколько  модулей в мавен в микросервисах  - все микросервисы в одном монорепо 
9. [ ] ДРУГИЕ БАЛАНСИРОВЩИКИ - API Gateway balancing , DNS Load Balancing ,  **Service Mesh balancing** 
10. [ ] в каких бизнес задачах обычно нужна балансировка ? 
11. [ ] ЗАДАЧА написать микросервисную систему в которой - 
12. [ ] при создание нового инстанса он подхватывает нужные ему настройки из конфига 
13. [ ] работает балансировка 
14. [ ] как балансировщик может отслеживать нагрузку на инстанс 
15. [ ] как откатить инстанс назад и как при этом должен работать пайплайн 
16. [ ] как произвести транзакции в БД на 2  инстанцах 
17. [ ] Spring Cloud Gateway 
# Напоминалки

>[!question]- как  запустить тестовый пример балансировки
>1. Запустить проект /home/igor/IdeaProjects/springdoc/eureka/EurekaServerApplication 
>2. Запустить проект /home/igor/IdeaProjects/springdoc/balancer/chatgpt/UserService 
>3. Запустить проект /home/igor/IdeaProjects/springdoc/balancer/chatgpt/OrderService через терминал и для каждого инстанса 
>   в отедльном терминале вызвать команду со своим портом , например для двух инстансов 
>   Терминал 1
>   ```
>    mvn spring-boot:run -Dspring-boot.run.arguments="--server.port=8080"
>   ```
>   Терминал 2
>   ```
>    mvn spring-boot:run -Dspring-boot.run.arguments="--server.port=8081"
>   ```
>4.  ввести в браузере http://localhost:8090/get-orders 
>5. Посмотреть все инстанцы можно здесть http://localhost:8761/ 

>[!question]-  у чего изолированный контекст 
> @LoadBalancerClient создает изолированный контекст Spring, и бины из главного контекста туда не попадают.

>[!question]- Чем отличается балансировка на стороне сервера от на стороне клиента 
>**Client-side (Spring Cloud LoadBalancer, Ribbon)  -** 
> - На стороне клиента (в коде) , 
> - кто зннает все о инстанцах - Клиент (через Eureka) 
> - Масштабируемость - требует чтобы все инстанцы могли балансировать (должны быть установлены зависимости )
> - **Каждый инстанс (экземпляр) сервиса должен зарегистрироваться в Eureka Server.** 
>  - Примеры - Spring Cloud LoadBalancer, Netflix Ribbon  
> **Server-side (Nginx, HAProxy, AWS ELB** 
> - РЕШЕНИЕ принимается На стороне балансировщика (в инфраструктуре)
> - знает о всех инстанцах Балансировщик 
> - масштабируемость  управляется Централизовано и проще администрировать
> - примеры Nginx, HAProxy, AWS ELB, Traefik 
> ДАЛЬШЕ

>[!question]- Где указывется конфигурация балансировки 
>@LoadBalancerClient(name = "OrderService", configuration = LoadBalancerConfiguration.class) 
>в классе LoadBalancerConfiguration а имя инстанса - OrderService 

>[!question]- Какая зависимость отвечает за балансировку нагрузки 
>Cloud Loadbalancer  
>```
><artifactId>spring-cloud-starter-loadbalancer</artifactId>
>```

>[!question]- какая аннотация делает бин балансирующем нагрузку
>```
> @LoadBalanced   
>```

>[!question]- класс отвечающий за балансировку 
>```
>@Configuration  
class RestTemplateConfig { 
>@Bean  
> @LoadBalanced    public RestTemplate restTemplate() {  
  >    return new RestTemplate();  
>}  
>}
>```
> п о умолчанию использет  алгоритм балансировки  Round Robin 

>[!question]- получить алгоритм балансировки через  .yml 
>```
spring:
cloud:
loadbalancer:
  clients:
order-service:
  configuration:
org.springframework.cloud.loadbalancer.core.RandomLoadBalancer
>```

>[!question]- Чем конфигурация микросервиса отличается от его настроек
>  Ribbon

>[!question]-  Config
>  Централизованное место управления  кофигурациями  (внешними свойстами приложений)
>  **Config Server** ты поднимаешь сам (это обычное Spring Boot приложение).  Может хранить конфиги в гит, игтхаб, файловой системе или в бзе 

>[!question]- Config -  server-side 
> @EnableConfigServer  - ПОСРЕДНИК МЕЖДУ МЕСТОМ ХРАНЕНИЯ И ПРИЛОЖЕНИЯМИ -  позволяет создать конфиг сервер спринга как отдельное приложение, которое служит  посредником между приложениями и гит системой

>[!question]- Config подключить к гиту
> application.preperties
> server.port=8888
> #spring.cloud.config.server.git.uri=git@github.com:NikitinGit/Obsidian-Vault-MAIN.git  
> spring.cloud.config.server.git.uri=https://github.com/NikitinGit/Obsidian-Vault-MAIN.git  
> #spring.cloud.config.server.git.uri=git@github.com:NikitinGit/Obsidian-Vault-MAIN.git  
> #spring.cloud.config.server.git.ignoreLocalSshSettings=false  
> #spring.cloud.config.server.git.privateKey=/home/igor/.ssh/nikitinssh  
> #spring.cloud.config.server.git.strictHostKeyChecking=false

>[!question]- Config refresh типы 
>   @RefreshScope + POST /actuator/refresh (текущий вариант) - требует ручной refresh после каждого изменения
  >2. Spring Cloud Bus - автоматическая рассылка изменений всем клиентам через message broker (RabbitMQ/Kafka). Один POST запрос на /actuator/bus-refresh обновит все клиенты
  > 3. Polling - клиент сам периодически проверяет изменения (можно настроить через spring.cloud.config.refresh-interval)
  > 4. Config Server webhooks - Config Server может отслеживать изменения в git и автоматически уведомлять клиентов

> [!question]- Config - а что обычно хранится в файлах конфига в микросервисной архитектуре  
> В файлах конфигурации микросервисов обычно хранится:  
> 1. Подключения к БД:  
>   - URL, credentials, pool settings  
>   - Параметры для разных окружений (dev/test/prod)  
>  
> 2. Внешние сервисы:  
>   - API endpoints других микросервисов  
>   - Timeouts, retry policies  
>   - Circuit breaker настройки  
>  
> 3. Инфраструктура:  
>   - Message brokers (Kafka, RabbitMQ)  
>   - Cache (Redis)  
>   - Service discovery (Eureka, Consul)  
>  
> 4. Безопасность:  
>   - JWT секреты, API keys  
>   - OAuth настройки  
>   - CORS настройки  
>  
> 5. Feature flags:  
>   - Включение/выключение функций  
>   - A/B тестирование  
>  
> 6. Логирование:  
>   - Уровни логов  
>   - Appenders  
>  
> 7. Business properties:  
>   - Лимиты, thresholds  
>   - Бизнес-параметры (например, комиссии, тарифы)  
>  
> НЕ хранят:  
>   - Код, бизнес-логику  
>   - Большие объемы данных  
>   - Чувствительные данные в открытом виде (используют vault/secrets manager)

>[!question]- Config -  client-side - почему важно указвать имя spring.application.name=a-bootiful-client иначе оно не работает ?
>   spring.application.name=ConfigClient
>   spring.cloud.config.name=a-bootiful-client
>    spring.cloud.config.label=main
>    spring.application.name=a-bootiful-client
>    spring.profiles.active=dev

>[!question]-  Spring Boot Actuator
> это библиотека и модуль для Spring Boot-приложений, который предоставляет готовые инструменты для мониторинга и управления приложением в режиме реального времени. 
> - Встроенные HTTP эндпоинты (например, `/actuator/health`, `/actuator/info`, `/actuator/metrics`) для проверки состояния приложения, информации и метрик.
> - Позволяет отслеживать здоровье приложения, собирать данные о работе JVM, нагрузке, использовании ресурсов и многом другом.
> - Позволяет управлять приложением через специальные точки доступа (например, перезапуск, изменение логирования).
> - Возможность включать/выключать или настраивать доступ к отдельным эндпоинтам.
> - Интеграция с системами мониторинга и алертинга (Prometheus, Grafana и др.).
> - Упрощает диагностику и контроль работы приложения в продакшене без необходимости писать много дополнительного кода.

>[!question]- JMX
>это 

>[!question]-  spring-cloud-netflix-server
>[Netflix Eureka service registry](https://github.com/spring-cloud/spring-cloud-netflix),
>сервер дискавери еврика,  через гет запрос который  атоматически (без перезагрузки) определяет какие серивисы работают  ```@EnableEurekaServer``` зависимость ```<dependency>  <groupId>org.springframework.cloud</groupId>  <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>  </dependency>```

>[!question]-  spring-cloud-netflix-client
>клиент секрвис, который может получать данные через сервер еврика от другого клиента сервиса  гет запрос  ```@EnableEurekaServer``` зависимость ```<dependency>  <groupId>org.springframework.cloud</groupId>  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId> </dependency>```
>Сам регистрируется в реестре еврика при старте
>Пингует сервер раз в 3 0 секунд на heartbeat 
>**периодически обновляют кэш** списка сервисов (по умолчанию раз в 30 секунд) 


>[!question]- Config -  client-side 
>  пиши

>[!question]- Config -  client-side 
>  пиши



# струтутра проекта 
● Отличный вопрос! В реальных компаниях используются оба подхода, в зависимости от размера команды, зрелости процессов и архитектуры. Вот как это выглядит:

  1. Monorepo (все в одном репозитории)

  Структура:

  company-backend/                    # Один Git репозиторий
  ├── user-service/
  │   ├── src/
  │   ├── pom.xml
  │   └── Dockerfile
  ├── order-service/
  │   ├── src/
  │   ├── pom.xml
  │   └── Dockerfile
  ├── payment-service/
  │   ├── src/
  │   ├── pom.xml
  │   └── Dockerfile
  ├── common/                         # Общие библиотеки
  │   ├── src/
  │   └── pom.xml
  ├── pom.xml                         # Родительский POM
  └── docker-compose.yml              # Для локальной разработки

  Используют:

  - Google (весь код в одном гигантском репозитории)
  - Facebook/Meta
  - Twitter
  - Uber
  - Многие стартапы и средние компании

  Преимущества:

  ✅ Проще рефакторинг - можно изменить API в одном коммите✅ Общий код легко переиспользовать✅ Атомарные изменения - один PR может менять несколько сервисов✅ Проще CI/CD - одна конфигурация для всех✅ Легче для новичков - всё в одном месте✅ Версионирование упрощается - все сервисы обновляются вместе

  Недостатки:

  ❌ Большой размер репозитория❌ Медленное клонирование❌ Сложные права доступа (все видят весь код)❌ Риск "большого взрыва" при деплое

  Пример конфигурации (Maven):

  <!-- Корневой pom.xml -->
  <project>
      <groupId>com.company</groupId>
      <artifactId>backend-parent</artifactId>
      <packaging>pom</packaging>

      <modules>
          <module>user-service</module>
          <module>order-service</module>
          <module>payment-service</module>
          <module>common</module>
      </modules>
  </project>

  ---
  2. Polyrepo (каждый сервис в своём репозитории)

  Структура:

  GitHub Organization: company-backend/
  ├── user-service/              # Отдельный репозиторий
  │   ├── src/
  │   ├── pom.xml
  │   ├── Dockerfile
  │   └── .git
  │
  ├── order-service/             # Отдельный репозиторий
  │   ├── src/
  │   ├── pom.xml
  │   ├── Dockerfile
  │   └── .git
  │
  ├── payment-service/           # Отдельный репозиторий
  │   ├── src/
  │   ├── pom.xml
  │   ├── Dockerfile
  │   └── .git
  │
  └── common-lib/                # Отдельный репозиторий (публикуется в Nexus/Artifactory)
      ├── src/
      ├── pom.xml
      └── .git

  Используют:

  - Netflix (пионеры микросервисов)
  - Amazon
  - Spotify
  - Крупные enterprise компании
  - Компании с множеством команд

  Преимущества:

  ✅ Полная изоляция сервисов✅ Независимое развёртывание✅ Чёткие границы ответственности команд✅ Гибкие права доступа (команда А не видит код команды Б)✅ Разные технологии (один на Java, другой на Go)✅ Меньше конфликтов в Git

  Недостатки:

  ❌ Сложнее рефакторинг между сервисами❌ Дублирование кода❌ Версионирование общих библиотек (нужен Nexus/Artifactory)❌ Сложнее CI/CD - нужно настраивать для каждого репозитория❌ Трудно отследить изменения между сервисами

  Пример работы с общими библиотеками:

  <!-- user-service/pom.xml -->
  <dependencies>
      <dependency>
          <groupId>com.company</groupId>
          <artifactId>common-lib</artifactId>
          <version>1.2.3</version>  <!-- Публикуется в Nexus -->
      </dependency>
  </dependencies>

  ---
  3. Гибридный подход (самый популярный)

  Структура:

  company-backend/                    # Monorepo для основных сервисов
  ├── core-services/
  │   ├── user-service/
  │   ├── order-service/
  │   └── payment-service/
  └── shared-libs/
      └── common/

  company-analytics/                  # Отдельный репозиторий для аналитики
  ├── analytics-service/
  └── reporting-service/

  company-mobile-backend/             # Отдельный репозиторий для мобильного API
  └── mobile-gateway/

  Критерии разделения:

  - По доменам (e-commerce, аналитика, админка)
  - По командам (команда А владеет репозиторием А)
  - По темпу изменений (часто меняющиеся vs стабильные сервисы)

  ---
  4. Реальная статистика

  По опросам разработчиков (State of Microservices 2023-2024):

  | Подход   | % компаний |
  |----------|------------|
  | Monorepo | 40%        |
  | Polyrepo | 35%        |
  | Гибрид   | 25%        |

  Тренды:
  - Стартапы → Monorepo (проще начать)
  - Средние компании → Гибрид
  - Enterprise → Polyrepo (строгая изоляция команд)
  - Big Tech → Monorepo (Google-style с продвинутыми инструментами)

  ---
  5. Инструменты для Monorepo

  Для управления большими monorepo используют:

  - Nx (для Node.js/Java)
  - Bazel (Google)
  - Pants (Twitter)
  - Maven Multi-Module (Java)
  - Gradle Multi-Project (Java/Kotlin)

  ---
  Что выбрать?

  Monorepo, если:
  - Маленькая команда (<20 человек)
  - Тесная связь между сервисами
  - Частые изменения API
  - Хотите простоту

  Polyrepo, если:
  - Большая компания (>100 человек)
  - Независимые команды
  - Разные технологии
  - Строгая изоляция

  Гибрид, если:
  - Средняя компания
  - Есть чёткие домены
  - Часть сервисов связаны, часть изолированы

  Вывод: Большинство современных компаний начинают с Monorepo, а при росте переходят на Гибридный подход.