# BASE

>[!question]- примитивы 
>byte, short, long, float, int, char, double, boolean  

>[!question]- JIT 
>JIT - just in time компиляция байткода при запуске программы - его горячих мест (hot spots) - которые часто используются

>[!question]- bytecode 
>bytecode - это набор инструкций для выполнения jvm, не зависит от ОС - промежуточный код 

>[!question]- JDK 
>JDK - набор разработчика содержит JRE и инструменты разраба, JRE - среда исполнения программы - достаточно для запуска проги - содержит jvm и необходимые компоненты/библиотеки, JVM ^jdk
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

