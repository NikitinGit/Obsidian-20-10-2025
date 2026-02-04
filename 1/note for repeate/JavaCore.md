# Задачи 
https://topjava.ru/blog/what-is-the-jre  
1. [x] the stack overflow exception - пример  
2. [x] как увеличить стек
3. [x] что хранится в куче 
4. [x] как увеличить кучу
5. [x] преобразjвание через (Myclass) может ли быть вверходящим - да , но в таком случае это преобразвание излишне 
6. [x] Методы object - допиши 11.49 - 12/05
7. [ ] Кастомизация чиловых типов
8. [ ] Function в стримах 
9. [ ] Consumer
10. [ ] BinaryOperator
11. [ ] Supplier
12. [ ] Зачем нежен hashCode() и как он создается, может ли он измениться -ДОПИШИ
13. [x] instanceof зачем пишут в equals переопределиние метода 
14. [ ] Методы object - посмотри в коде 12.55 -14.37
15. [x] иквелс (у всех насдеыдников одина ссылка на объект )
16. [x] тостринг
17. [x] instanceof что делает 
18. [ ] hashCode
19. [ ] clone
20. [ ] getClass
21. [ ] notify
22. [ ] notifyAll
23. [ ] wait
24. [ ] дженерики - ковариантность, инвариантность , контвариантность 
25. [ ] Методы рефлексии
26. [ ] как Thread.currentThread() связан с Object
27. [ ] мониторинг программ 
28. [ ] Посмотри лямбда в стрим апи линекд хеш мап 
29. [ ] Почему при создании хешмап сбивается сортировка   

# JVM
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

# BASE
>[!question]- while(test()); может быть ?
>Да

>[!question]- запустить отдельный класс в программе в терминале через мавен
>cd /home/igor/IdeaProjects/TestLinux
>mvn exec:java -Dexec.mainClass="com.example.testlinux.blind.seal.BlindSeal"
>чтобы обновить приложение надо сделать mvn clean compile - при этом надо находится в папек где расположен .pom

>[!question]- Классы для интерактивного приложения - способы читать ввод из командной строки 
> Scanner scanner = new Scanner(System.in); 
> String input = scanner.nextLine();
> char inputChar = input.charAt(0); 
> без ентер - raw mode
> ```
> try (Terminal terminal = TerminalBuilder.builder().system(true).build())
> terminal.enterRawMode();
> NonBlockingReader reader = terminal.reader();  
> terminal.writer().println("Нажмите Backspace для выхода\n");  
> terminal.flush();
> ```

>[!question]- Set.of() когда вызывает исключения 
>при дублях если есть 

>[!question]- методы Object
>```
>public final Class<?> getClass() Возвращает объект класса `Class`, представляющий реальный тип объекта.
>```
>```
>public final void wait() throws InterruptedException - Останавливает текущий поток до вызова `notify()` / `notifyAll()`. 
>```
> ```
> public boolean equals(Object obj) - Сравнивает два объекта **на логическое равенство**, не на одинаковость ссылок. 
> ```
> ```
> public String toString() - представление в строковом виде
> ```
> ```
> public int hashCode() - Возвращает хэш-код объекта. 
> ```
> ```
> public final void wait(long timeout) throws InterruptedException
> public final void wait(long timeout, int nanos) throws InterruptedException
>  перегрузка метода с таймаутом 
> ```
> ```
> public final void notify() - Пробуждает один ожидающий поток. 
> ```
> ```
> public final void notifyAll() - Пробуждает все ожидающие потоки. 
> ```
> Protected методы (доступны в наследниках) 
> ```
> protected Object clone() throws CloneNotSupportedException - Создаёт копию объекта (поверхностное клонирование). 
> ```
> ```
>protected void finalize() throws Throwable (устарел) - Вызывался перед удалением объекта GC.  
> С JDK 9 помечен как deprecated → нельзя применять.
> ```

>[!question]- Почему нельзя нарушать контракт **equals + hashCode** 
>хэш коллекции сравнивают элементы по хэш коду 

>[!question]- super вызывает что 
>конструктор или метод ближайшего родителя 
>Java НЕ СМЕНЯЕТ тип объекта при вызове `super.getObject()`. 

>[!question]- обязательно ли писать super() в конструкторе наследника
> Да если не объявлен конструктор по умолчанию , нет если объявлен.
> Компитор сам допишет его в первом случае 
> Если объявлен конструктор с параметрами - то обязательно надо объявить конструктор без параметров 

>[!question]- что делает instanceof 
>сравнивает фактический тип объекта в памяти (runtime type), а НЕ тип переменной (reference type). 
>То есть у базового класса все его наследники имеют один тип, который может называться любым классом от бащвого то последнего наследника (GrandParrent Parent Child )
>Например 
>```
>o instanceof SomeClass 
>```
>говорит 
>«Является ли объект `o` экземпляром класса `SomeClass` или его наследником?» 
> У  объекта всегда один тип на всех наследников - он равен любому из типов наследников и родителей 

>[!question]- меняет ли объект  родителя его кастомизация к наследнику 
> Нет , пример 
> ```
> PrivateOverride po = new Derived(); 
> Derived test = (Derived)po; 
> System.out.println("PrivateOverride test equals Derived test1; " + test.equals(po));
> output; true
> ```

>[!question]- Какая кастомизация разрешается а какая нет 
> Разрешается 
> ```
> PrivateOverride po = new Derived(); 
> Derived test = (Derived)po; 
> ```
> Не разрешается 
> ```
> PrivateOverride po = new PrivateOverride(); 
> Derived test = (Derived)po; 
> ```
> Привести объект можно **только к его реальному типу или одному из его родителей**. 

>[!question]- преоброзование через (Myclass) может ли быть вверходящим (от предка к родителю)
> Да , но в таком случае это преобразвание излишне 
> ```
> PrivateOverride po = new PrivateOverride();
> Derived derived = new Derived();
> PrivateOverride test = (PrivateOverride)derived;
> ```

>[!question]-  Как проверить тип  объекта 
>if (bean instanceof MyBean) 

>[!question]- полиморфизм
>способность базового класса принимать  форму своих наследников, но не наоборот
>например так нельзя 
>```
>PrivateOverride po = new PrivateOverride();
>Derived test1 = (Derived) po; //ERROR
>output; Exception in thread "main" java.lang.ClassCastException: class com.example.testlinux.polymorphism.PrivateOverride cannot be cast to class com.example.testlinux.polymorphism.Derived  
>```
>но так можно
>```
> Derived derived = new Derived();
> PrivateOverride test = (PrivateOverride)derived;  
> Derived test1 = (Derived) test;
>```
> потому что даже не смотря на преоброзование (PrivateOverride) test все равно принимает форму  Derived 
>Скрытые  или final  поля базового класса не переопределяются в наследниках 

>[!question]- сколько публичных классов может быть в одном файле 
>один

>[!question]- примитивы 
>boolean -  определяется jvm 
>byte = 8 бит , -128  до 127
>short, 16 бит ,   -32 768 … 32 767
>char,  - 16 бит , символ юникод
>  int,  = 32бита , −2 147 483 648 … 2 147 483 647
>  float - 32 бит ~ ±3.4 × 10³⁸
> long - 64 бит  −9 223 372 036 854 775 808 … 9 223 372 036 854 775 807 
>  double   - 64 бит  ~ ±1.7 × 10³⁰⁸ 

>[!question]- JIT 
>JIT - just in time компиляция байткода при запуске программы - его горячих мест (hot spots) - которые часто используются

>[!question]- bytecode 
>bytecode - это набор инструкций для выполнения jvm, не зависит от ОС - промежуточный код 

>[!question]- JDK 
>JDK - набор разработчика содержит JRE и инструменты разраба, 

>[!question]- JRE 
>JRE - среда исполнения программы - достаточно для запуска проги - содержит jvm и необходимые компоненты/библиотеки 
>Набор стандартных библиотек Java (например, java.util, java.io, java.awt и др.)
>Верификатор байткода, проверяющий корректность и безопасность загружаемого кода.
>Интерпретатор, который читает и выполняет байт-код построчно.
>JRE не включает компилятор Java

>[!question]- Память в java 
>делится на 3 части - стэк, куча и метаспейс (не изменная часть проги - например где хранятся классы )
>the stack overflow exception - переполненеи стека  - Как правило, переполнение стека происходит когда метод или методы обращаются друг к другу зацикленным способом, тем самым выделяя постоянно растущее число вызовов в стек. 

>[!question]- Что хранит стек
> локаьлные перемекнные метода 
> параметры метода 
> адрес  возврата 
> служебные данные JVM. 
> все это хранится в stack frame 
> когда метод заканчивается — фрейм удаляется (pop). 
> ```
> Локальная переменная (ссылка) в стеке исчезает раньше,  
чем бъект, на который она указывает, освобождается из кучи.
> ```

>[!question]-  Когда вызызвается toString()
> System.out.println(obj);  

 >[!question]- stack overflow exception когда возникает 
 >При рекурсии 
 
  >[!question]- хвостовая рекурсия 
 > Не ждет вызова остальных методов
 
   >[!question]- увлечить стек 
java -Xss512k TestLinuxApplication - 512  кб
 mvn spring-boot:run -Dspring-boot.run.jvmArguments="-Xss512k"  
 **IntelliJ IDEA** →  
`Run > Edit Configurations > VM options`  
→ впиши `-Xss2m`
java -Xss1m TestLinuxApplication - с Мб

>[!question]- узнать глубину текущего стектрейса 
>Thread.currentThread().getStackTrace().length 

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

>[!question]- установить размер кучи 
>java -Xms256m -Xmx2g MyProgram

# лямбда 

>[!question]-  что работает быстрее  - .forEach или просто for 
>```
>bidFighterList.forEach(bidFighter -> {
>bidFighter.setApproved(newStatus.num);
>bidFighter.setApproved(BidStatus.APPROVED.num);  
});
>```
> .forEach работает медленнее потому что  использует лямбд-выражение, то есть создает объект функционального интерфейса (Consumer)
> из за вызова скрытого мтеода accept()
> **.foreach Указывает  но то, что внутри должны менятся тольк оего элементы, но есть исеключения*
> фунциональный метод 
> break |continue  не используются 

>[!question]- как создать int[]
>int[] test = new int[]{1, 2};
>int[] test2 = new int[25];
>int[] test3 = {2, 5};

>[!question]- сделать из  int[] nums - Map
>boxed() превращает int  в Integer
>```
>Map<Integer, Integer> mapOfNums = IntStream.range(0, nums.length)
.boxed().collect(Collectors.toMap(i -> i, i -> nums[i]));
>```
>```
>Map<Integer, Integer> mapOfNums = new HashMap<>(); Arrays.stream(nums).forEach(n -> mapOfNums.put(n, nums[n]));
>```
> отсортировать по значению, без LinkedHashMap::new не сортируется ? 
> ```
> Map<Integer, Integer> mapOfNums = IntStream.range(0, nums.length).boxed()  
>.sorted(Comparator.comparingInt(i -> nums[i]))  
>.collect(Collectors.toMap(i -> i, i -> nums[i], (a, b) -> a, LinkedHashMap::new));
> ```

>[!question]- получить значение мап по индексу
>```
>Integer i2 = mapOfNums.entrySet().stream().filter(e -> value == e.getValue())
.map(Map.Entry::getKey)
.findFirst().orElse(null);
>```

>[!question]- метод проверки существования ключа/значения Map
>containsKey
>containsValue 

>[!question]- длинна массива int[]
>длинна массива array = mums.length 

>[!question]- отличие интерфейса от абстрактного класса
>у интерфейса не может быть состояния, у абстрактного класса может через поля его

>[!question]- Optional это 
контейнер который  может быть пустым




# 👑 Итог — большая истина JVM

### 👉 Объект всегда одного типа — того, что был создан через `new`.

### 👉 `this` всегда указывает на реальный объект (самый нижний класс).

### 👉 Родительские методы, вызванные на объекте наследника, работают с this = наследник.

### 👉 instanceof проверяет только реальный тип объекта.

### 👉 Тип переменной не влияет ни на что — это всего лишь “ярлык”.

