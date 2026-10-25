# MapStruct

>[!question]- Что такое MapStruct и зачем он нужен
>**MapStruct** — генератор мапперов: по интерфейсу вида `BidResponse toResponse(Bid bid)` **при компиляции** пишет класс, который перекладывает поля сущности в DTO (и обратно).
>Зачем: убрать ручное перекладывание полей и получить **проверку при сборке** — забыл поле в маппинге → проект не собирается.
>
>| Способ | Плюсы | Минусы |
>|---|---|---|
>| Руками (`static from(Bid bid)`) | явно, без магии | при добавлении поля легко забыть его здесь |
>| **MapStruct** | генерируется при компиляции, быстро как ручной код, с `unmappedTargetPolicy=ERROR` забытое поле — ошибка сборки | ещё один процессор аннотаций; с Lombok нужна связка `lombok-mapstruct-binding` |
>| DTO прямо из запроса (`select new ...Dto(b.id, ...)` в JPQL) | БД отдаёт только нужные колонки | только для чтения |
>| ModelMapper | ноль кода | рефлексия во время работы: ошибки видны только в рантайме, медленнее |
>
>Сущность наружу вместо DTO не отдают: изменение таблицы молча меняет API, а ленивые связи ломают сериализацию / вызывают N+1.

>[!question]- Что нужно, чтобы MapStruct сгенерировал маппер — 3 шага
>**1. Подключить в `pom.xml`** — это **две разные** библиотеки:
>```xml
><!-- аннотации (@Mapper, @Mapping), которые пишешь в коде -->
><dependency>
>    <groupId>org.mapstruct</groupId>
>    <artifactId>mapstruct</artifactId>
>    <version>${mapstruct.version}</version>   <!-- Spring Boot версией НЕ управляет — задать в <properties> -->
></dependency>
>```
>```xml
><!-- в maven-compiler-plugin: генератор, работает только при компиляции, в jar не попадает -->
><annotationProcessorPaths>
>    <path><groupId>org.projectlombok</groupId><artifactId>lombok</artifactId><version>${lombok.version}</version></path>
>    <path><groupId>org.mapstruct</groupId><artifactId>mapstruct-processor</artifactId><version>${mapstruct.version}</version></path>
>    <!-- чтобы MapStruct видел геттеры/конструкторы, сгенерированные Lombok -->
>    <path><groupId>org.projectlombok</groupId><artifactId>lombok-mapstruct-binding</artifactId><version>0.2.0</version></path>
></annotationProcessorPaths>
>```
>Без `mapstruct-processor` проект соберётся, но `BidMapperImpl` не появится → Spring при старте: «нет бина `BidMapper`».
>
>**2. Сделать маппер Spring-бином** (для всех мапперов сразу):
>```xml
><compilerArgs>
>    <arg>-Amapstruct.defaultComponentModel=spring</arg>   <!-- над сгенерированным классом будет @Component -->
>    <arg>-Amapstruct.unmappedTargetPolicy=ERROR</arg>     <!-- незаполненное поле DTO = ошибка сборки -->
></compilerArgs>
>```
>Или в каждом маппере: `@Mapper(componentModel = "spring")`. Без этого реализацию получают вручную: `Mappers.getMapper(BidMapper.class)`.
>
>**3. Описать интерфейс:**
>```java
>@Mapper
>public interface BidMapper {
>    BidResponse toResponse(Bid bid);
>}
>```
>И внедрять как обычный бин: `private final BidMapper bidMapper;` + `@RequiredArgsConstructor`.

>[!question]- Как MapStruct сопоставляет поля
>По сигнатуре метода понимает **из чего** (`Bid`) и **во что** (`BidResponse`), а поля сопоставляет **по именам**: у `BidResponse` есть `eventId` → ищет в `Bid` `getEventId()` → подставляет.
>Record заполняется через его конструктор, обычный класс — через сеттеры или конструктор.
>
>Если имена разные или поле заполняется не из источника — указывают явно:
>```java
>@Mapping(target = "fighter", source = "fighterId")               // другое имя
>@Mapping(target = "comment", ignore = true)                      // не заполнять
>@Mapping(target = "title", expression = "java(bid.getEventId() + \"-\" + bid.getFighterId())")  // вычислить
>BidResponse toResponse(Bid bid);
>```
>Список: `List<BidResponse> toResponses(List<Bid> bids);` — MapStruct сам пройдёт по элементам через `toResponse`.

>[!question]- Когда и где появляется реализация
>При **каждой компиляции** (`./mvnw compile`, Build → Rebuild Project):
>`target/generated-sources/annotations/.../BidMapperImpl.java` — в исходниках его нет и в git он не попадает.
>Сгенерированный код — то же, что написал бы руками:
>```java
>@Component
>public class BidMapperImpl implements BidMapper {
>    public BidResponse toResponse(Bid bid) {
>        if (bid == null) return null;
>        ...
>        return new BidResponse(bid.getId(), bid.getEventId(), ...);
>    }
>}
>```
>Проверка защиты: добавил в `BidResponse` поле `comment`, которого нет в `Bid` → сборка падает:
>```
>BidMapper.java:[11,17] Unmapped target property: "comment".
>```

>[!question]- IntelliJ: «BidMapperImpl не найден» / «No qualifying bean of type BidMapper»
>Почти всегда одно из:
>1. **Load Maven Changes** (Ctrl+Shift+O) — IDE не подхватила процессоры из `pom.xml`.
>2. **Build → Rebuild Project** — реализация не перегенерирована после правки интерфейса.
>3. Settings → Build → Compiler → Annotation Processors → **Enable annotation processing**.
>4. Крайний случай: удалить кэш `rm -rf ~/.m2/repository/org/mapstruct/` и пересобрать.
>
>Массовые ошибки «метод/конструктор не найден» в сгенерированных `*MapperImpl` = IDE собрала проект **без Lombok** (не видит геттеры) → проверить порядок процессоров и `lombok-mapstruct-binding`.
