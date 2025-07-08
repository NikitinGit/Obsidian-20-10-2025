## **1. JVM и производительность**

- [[JUN#^jvm|Как работает JVM? Из чего состоит?]]
- Что такое **JIT-компилятор**?
- Как работает **Garbage Collector (GC)**? Какие есть алгоритмы сборки мусора?
- Что такое **Heap и Stack** в Java?
- Различия между **Minor GC, Major GC и Full GC**.
- Что такое **OutOfMemoryError (OOM)**? Как его избежать?
- Как профилировать и оптимизировать Java-приложение?
- Как работает `SoftReference`, `WeakReference`, `PhantomReference`?

---

## 🔹 **2. Коллекции (Collections Framework)**

- Как устроены **ArrayList, LinkedList, HashMap, HashSet, TreeSet**?
- Разница между **HashMap и ConcurrentHashMap**.
- Как работает метод **hashCode()**? Что произойдет, если у двух объектов одинаковый hashCode?
- Как работает **equals() и compareTo()**?
- Как реализовать свой **Immutable Collection**?
- Чем отличается **synchronizedList() от CopyOnWriteArrayList**?

---

## 🔹 **3. Многопоточность (Concurrency)**

- Разница между **Thread, Runnable и Callable**.
- Как работает **synchronized**? Чем отличается от **ReentrantLock**?
- Что такое **volatile**? Где его использовать?
- Разница между **wait(), notify(), notifyAll()**.
- Что такое **ForkJoinPool** и **CompletableFuture**?
- Как работает **ExecutorService**? Чем отличается от `ThreadPoolExecutor`?
- Разница между **CountDownLatch, CyclicBarrier и Phaser**.
- Что такое **CAS (Compare-And-Swap)**?
- Как устроены **ConcurrentHashMap, CopyOnWriteArrayList**?
- Что такое **ThreadLocal** и когда его использовать?

---

## 🔹 **4. Исключения и обработка ошибок**

- Чем отличаются **checked и unchecked exceptions**?
- Что лучше: `throws Exception` или `try-catch`?
- Как правильно использовать `try-with-resources`?
- Как реализовать **собственное исключение**?
- Как обрабатывать ошибки в многопоточной среде?

---

## 🔹 **5. Паттерны проектирования (Design Patterns)**

- Что такое **SOLID**? Как их применять?
- Разница между **Factory, Abstract Factory, Singleton**.
- Что такое **Builder, Prototype, Decorator**?
- Что такое **Proxy, Strategy, Observer**?
- Разница между **Monolithic, SOA и Microservices**.
- Принципы **DDD (Domain-Driven Design)**.
- Что такое **CQRS (Command Query Responsibility Segregation)**?

---

## 🔹 **6. Spring Framework (Spring Boot)**

- Как работает **Dependency Injection (DI)**?
- Разница между `@Component`, `@Service`, `@Repository`.
- Что такое `@Transactional`? Как работает Spring Transaction Management?
- Как работает **Spring AOP (Aspect-Oriented Programming)**?
- Разница между `@RestController` и `@Controller`.
- Как работает **Spring Security**? Какие способы аутентификации?
- Как настраивать **Spring Boot Starter**?
- Как работает **Spring WebFlux (Reactive Programming)**?
- Что такое **Spring Actuator**?

---

## 🔹 **7. Hibernate и работа с базами данных**

- Разница между **JDBC и Hibernate**.
- Что такое **Hibernate Session и EntityManager**?
- Разница между `@OneToOne`, `@OneToMany`, `@ManyToMany`.
- Как работает **Lazy Loading и Eager Loading**?
- Что такое **N+1 проблема**? Как её решать?
- Как работает **Hibernate Cache (1-го и 2-го уровня)**?
- Как написать **Native Query в Hibernate**?
- Чем отличается **ACID от BASE**?
- Разница между **SQL и NoSQL**?

---

## 🔹 **8. REST API и микросервисная архитектура**

- Чем отличается **REST от SOAP**?
- Как правильно верстать **REST API (HATEOAS, RESTful Best Practices)**?
- Что такое **HTTP методы: GET, POST, PUT, DELETE, PATCH**?
- Как работает **OAuth2 и JWT**?
- Что такое **Circuit Breaker (Resilience4j, Hystrix)**?
- Что такое **API Gateway (Kong, Zuul, Nginx)?**
- Как реализовать **Rate Limiting**?

---

## 🔹 **9. Kafka, RabbitMQ, Message Brokers**

- Что такое **Kafka и RabbitMQ**?
- Разница между **Push-based и Pull-based** подходами?
- Как работает **Kafka Consumer и Producer**?
- Что такое **Partition, Offset, Consumer Group**?
- Как обеспечить **Exactly Once Delivery**?
- Redis

---

## 🔹 **10. DevOps и CI/CD**

- Как работает **Docker**? Что такое Dockerfile?
- Как деплоить приложение в **Kubernetes**?
- Что такое **CI/CD (Jenkins, GitHub Actions, GitLab CI)**?
- Что такое **Logging & Monitoring (ELK Stack, Prometheus, Grafana)**?
- Как настраивать **Spring Boot с Kubernetes**?