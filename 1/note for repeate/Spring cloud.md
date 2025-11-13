# Задачи 
https://chatgpt.com/c/6915e111-75d4-832a-acc4-e6004a51044c 
1. [x] Чем отличается балансировка на стороне сервера от на стороне клиента 
2. [ ] ДРУГИЕ БАЛАНСИРОВЩИКИ - API Gateway balancing , DNS Load Balancing ,  **Service Mesh balancing** 
3. [ ] как еврика или конфиг или что то еще может определить насколько сильно забит инстанс запросами и как сильно занят вычислениями ? что в спринг за это отвечает ?
4. [ ] ЗАДАЧА написать микросервисную систему в которой - 
5. [ ] при создание нового инстанса он подхватывает нужные ему настройки из конфига 
6. [ ] работает балансировка 
7. [ ] как балансировщик может отслеживать нагрузку на инстанс 
8. [ ] как откатить инстанс назад и как при этом должен работать пайплайн 
9. [ ] как произвести транзакции в БД на 2  инстанцах 
10. [ ] Spring Cloud Gateway 
# Напоминалки

>[!question]-  изолированный контекст 
> @LoadBalancerClient создает изолированный контекст Spring, и бины из главного контекста туда не попадают.

>[!question]- Чем отличается балансировка настороне сервера от на стороне клиента 
>Client-side (Spring Cloud LoadBalancer, Ribbon)  - 
> - На стороне клиента (в коде) , 
> - кто зннает все о инстанцах - Клиент (через Eureka) 
> - Масштабируемость - требует чтобы все инстанцы могли балансировать (должны быть установлены зависимости ? )
>  - Примеры - Spring Cloud LoadBalancer, Netflix Ribbon  
>Server-side (Nginx, HAProxy, AWS ELB 
> - РЕШЕНИЕ принимается На стороне балансировщика (в инфраструктуре)
> - знает о всех инстанцах Балансировщик 
> - масштабируемость  управляется Централизовано и проще администрировать
> - примеры Nginx, HAProxy, AWS ELB, Traefik 
> ДАЛЬШЕ

>[!question]- где указывется конфигурация 
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
>Мингует сервер раз в 3 0 секунд на heartbeat 
>**периодически обновляют кэш** списка сервисов (по умолчанию раз в 30 секунд) 


>[!question]- Config -  client-side 
>  пиши

>[!question]- Config -  client-side 
>  пиши

