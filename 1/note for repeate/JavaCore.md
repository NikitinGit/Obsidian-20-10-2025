# Задачи 
https://topjava.ru/blog/what-is-the-jre  
1. [x] the stack overflow exception - пример  
2. [x] как увеличить стек
3. [x] что хранится в куче 
4. [x] как увеличить кучу
5. [x] преобразjвание через (Myclass) может ли быть вверходящим - да , но в таком случае это преобразвание излишне 
6. [ ] Методы object - допиши и  посмотри в коде 
7. [ ] Методы рефлексии
8. [ ] как Thread.currentThread() связан с Object
9. [ ] мониторинг программ 
10. [ ] Посмотри лямбда в стрим апи линекд хеш мап 
11. [ ] Почему при создании хешмап сбивается сортировка   

# BASE

>[!question]- методы Object
>```
>public final Class<?> getClass() Возвращает объект класса `Class`, представляющий реальный тип объекта.
>```
>```
>public final void wait() throws InterruptedException - Останавливает текущий поток до вызова `notify()` / `notifyAll()`. 
>```
> equals ()- соответствие по ссылке
> toString() - представление в строковом виде
> notify()
> notifyAll()
> hashCode()

>[!question]- преоброзование через (Myclass) может ли быть вверходящим (от предка к родителю)
> Да , но в таком случае это преобразвание излишне 

>[!question]- проверить типа  объекта 
>if (bean instanceof MyBean) 

>[!question]- полиморфизм
>способность базового класса принимать  свою версию в производных классах
>скрытые  или final  поля базового класса не переопределяются в наследниках 

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
>JDK - набор разработчика содержит JRE и инструменты разраба, JRE - среда исполнения программы - достаточно для запуска проги - содержит jvm и необходимые компоненты/библиотеки, JVM 

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

>[!question]-  Методы object 

>[!question]-  Когда вызызвается toString()
> System.out.println(obj);  

 >[!question]- stack overflow exception когда возникает 
 >При рекурсии 
 
  >[!question]- хвостовая рекурсия 
 > не ждет вызвоа осталдьных методов
 
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

# функциональные методы интерфейсы

Function
Consumer
BinaryOperator
Supplier

