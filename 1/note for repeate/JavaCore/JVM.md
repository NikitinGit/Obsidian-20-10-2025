1. [ ] типы jvm - open jdk, ...
2. [ ] допиши когда какой сборщик мусора использовать https://www.perplexity.ai/search/jvm-kogda-kakoi-sborshchik-isp-cbnJ27T3TBWH6Bhdkd3zFg 


>[!question]- JVM это 
> виртуальная машина , которая интерпретирует байт-код, с помощью JIT компилирует его в машинный код. Jit анализирует код и находит в нем горячие участки кода - которые чаще всего исполняются и компилирует их в машинный код ускоряя выполнение программы. 
   Существует 2 вида JIT -  C1 (3 уровня компиляции - флаг -client) и  C2 (4-ый уровень компиляции - флаг -server). Клиентский быстрее запускает но работает медленнее. 
   На ОС x64  флаг -server отсутствует

>[!question]- `classpath` — это 
>**список путей**, где JVM ищет _скомпилированный байткод_ (`.class`-файлы).  Он **не хранит** байткод, а просто **указывает, где его искать** необходимые классы и ресурсы для запуска или компиляции Java-приложений.
>В командной строке при запуске Java-программы с помощью параметра `-cp` или `-classpath`. Например: 
>```
>java -cp /path/to/classes:/path/to/lib.jar com.example.Main
>```
>Example 
>```<?xml version="1.0" encoding="UTF-8"?>
<classpath>
<<classpathentry kind="src" path="src"/>
< <classpathentry kind="con" path="org.eclipse.jdt.launching.JRE_CONTAINER"/>
 <  <classpathentry kind="lib" path="lib/mysql-connector-j-8.0.33.jar"/>
<<classpathentry kind="lib" path="lib/gson-2.10.jar"/>
<    <classpathentry kind="output" path="bin"/>
</classpath>
>```

>[!question]-  компиляция 
> преолразования кода в байт код , используется javac  и его аналоги в jdk
> обратный процесс - декомпиляция 


>[!question]- что хранит куча
>объекты, строки, коллекции , массивы , все что создается через new 
>пример 
>``` 
>public class Example {
>public static void main(String[] args) {
>    int a = 10;                       // локальная переменная → стек
>        String s = "hello";               // строковый литерал → специальная область (String pool)
>            int[] arr = new int[5];           // сам объект массива → куча
>            User user = new User("Alice");    // объект User → куча
 >   }
>}
>```

   >[!question]- увлечить стек 
java -Xss512k TestLinuxApplication - 512  кб
 mvn spring-boot:run -Dspring-boot.run.jvmArguments="-Xss512k"  
 **IntelliJ IDEA** →  
`Run > Edit Configurations > VM options`  
→ впиши `-Xss2m`
java -Xss1m TestLinuxApplication - с Мб

>[!question]- установить размер кучи 
>java -Xms256m -Xmx2g MyProgram