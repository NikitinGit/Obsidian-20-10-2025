# MAVEN

1. [ ] модули 
2. [ ] плагины
3. [ ] жизненный цикл мавен 
4. [ ] установить на впн
5. [ ] ci/cd
6. [x] отличиие плагинов от зависимостей в мавен
7. [ ] создать своб зависимость и добавить в другой проект 
8. [ ] создать свой плагитн и доабьвить в другой рпоект 

>[!question]- отличие от ант и градле  
> Ант - Является наиболее «низкоуровневым» и гибким инструментом. Процесс сборки описывается в XML-файле build.xml, где вручную прописываются все задачи и шаги.  - Для управления зависимостями необходимо использовать дополнительные инструменты (например, Ivy).
> Градл (Gradle):  - Использует скриптовый DSL (на Groovy или Kotlin), что даёт большую гибкость и возможность программирования процесса сборки.
> Мавен - - Основывается на стандартизированной структуре проекта и управлении зависимостями из-за централизованного файла pom.xml.

>[!question]- плагины 
>то , что собирает проект, зависимости попадают в класпатч- плагины нет 

>[!question]- запустить приложение  на определенном порту 
> mvn spring-boot:run -Dspring-boot.run.arguments="--server.port=8082" 

>[!question]- как создавать модули  
> 

>[!question]- Интеграция с CI/CD*
> 

>[!question]- на чем написан 
> java, но может использоваться на ruby, scala 

>[!question]-  ```dependencyManagement```  это
> **специальный раздел** в `pom.xml`, который используется для **централизованного управления версиями зависимостей** в Maven-проекте
> например в этом блоке  мавен запомнит версию и далее можно её не писать 
> ```
> <dependencyManagement>
  >  <dependencies>
>        <dependency>
  >          <groupId>com.fasterxml.jackson.core</groupId>
>            <artifactId>jackson-databind</artifactId>
  >          <version>2.17.2</version>
>        </dependency>
  >  </dependencies>
></dependencyManagement>
> ```
> и  далее можно прописать так 
> ```
><dependency>
>    <groupId>com.fasterxml.jackson.core</groupId>
  >  <artifactId>jackson-databind</artifactId>
></dependency>
> ```
> — **версия автоматически подставится из `dependencyManagement` (2.17.2)**,  и не нужно явно её указывать.
> `<type>pom</type>` Говорит Maven, что мы импортируем _pom-файл_, а не jar-библиотеку 
|`<scope>import</scope>` Разрешает Maven “влить” (`import`) все определения из этого pom-а (обычно BOM) в твой `<dependencyManagement>` Maven буквально “вклеивает” всё это в твой `<dependencyManagement>`. 

>[!question]- `<version>${spring-cloud.version}</version>`  это 
>использует версию , указанную в 
>```
><properties>  
<spring-cloud.version>2025.0.0</spring-cloud.version>  
</properties>
>```

>[!question]- exclusions
>исключение автоматически подтягиваемых зависимостей , например 
>```
><dependency>> <groupId>org.springframework.boot</groupId>><artifactId>spring-boot-starter-web</artifactId>
><exclusions>
>        <exclusion>
>            <groupId>com.jayway.jsonpath</groupId>
 >           <artifactId>json-path</artifactId>
 >       </exclusion>
 >   </exclusions>
></dependency>
<dependency>
><groupId>com.jayway.jsonpath</groupId>
><artifactId>json-path</artifactId>
><version>2.9.0</version>
></dependency>
>```


>[!question]- Зависимости это 
> пиши

>[!question]- properties это 
> секция в файле `pom.xml`, предназначенная для **централизованного объявления переменных**, которые можно повторно использовать в других частях POM

>[!question]- обновить проект со сбросом кеша
>mvn clean install -DskipTests -U  

>[!question]- запустить спринг проект
>mvn spring-boot:run

>[!question]- узнать путь в wsl где установлен мавен 
>which mvn


