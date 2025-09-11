
https://metanit.com/java/tutorial/10.1.php 

>[!question]- BASE
>```
>1. В основе лежат промежуточные операции и терминальные (после которых промежуточные не могут быть вызваны, так же как терминальные не могут быть вызваны)
>2. Промежуточных операций может быть несколько - терминальная операция одна Использует отложение выполнение лямбда выражений - то есть при вызове терминальной операции 
 >3. Во всех коллекциях начиная с jdk 8 есть метод stream через который можно получать объект Stream<T> , так же его можно получить через Arrays.stream(T[] array), Stream.of("Nikitin", "Bin") , InStream.of() , LongStream.of(), DoubleStream.of() ```
>[!question]- Отличие стрим потока от списка элементов
>Пиши

>[!question]- сделать из списка ассоциативную коллекцию 
>Не убирая дубликаты
>4. 
>```
>Map<Integer, Fighter> mapObjById = list.stream().collect(      Collectors.toMap(Fighter::getId, o -> o));
>```
>```
>Map<Integer, Fighter> mapObjById = list.stream().collect(      Collectors.toMap(Fighter::getId,  Function.identity()));
>```
>Убрать дубликаты и оставить первое значение
>```
>Map<Integer, Fighter> mapObjById = list.stream().collect(Collectors.toMap(Fighter::getId, o -> o, (f1, f2) -> f1));
>```

>[!question]- какими могут быть лямбда выражения
>what

>[!question]- примеры терминальных операций 
>count()
>collect()
>toList()
>дальше пиши

>[!question]- Отложенное выполнение
>промежуточные вычисления вызываются при вызове терминальных

>[!question]- Ленивые вычисления 

