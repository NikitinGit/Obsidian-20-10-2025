https://www.google.com/search?q=stream+api+%D1%87%D1%82%D0%BE+%D0%BC%D0%BE%D0%B6%D0%B5%D1%82+%D0%B1%D1%8B%D1%82%D1%8C+%D0%B2+map&sca_esv=ad175a4a610f2e0b&sxsrf=ANbL-n55KnNEpQdNdLR6Pp5fStb1RH6Plg%3A1772653941597&fbs=ADc_l-Z6juH7r_7-8pNw5rPlE-L7lKX-xQDzwJouywX5Qr83x-wndlrxiB7XC753aUrOCsMw_45F-xS_9DgnjEXKZHlq78b_swAkVmiHWTCyU-0GSmWRc_F6uB_HEXgnPveaPwqIUcS9wvEHUBIrP4qPxk5VBIAyr2Ii1OzZ7GxHhmM6XWbL_azspV9xM1R2XVnXFULq1C1PD5KqHp6JuSgk9GQt-jCEl-Z8e6Nb2mWFC0vBoOfZuIKPpjRg_BYNOfa_4lu_lAN00vXHlhZfGIbwQ0erZXBjDA&aep=1&ntc=1&sa=X&ved=2ahUKEwjCnf7DgoeTAxVz_7sIHQKsB3IQ2J8OegQIFxAE&biw=2560&bih=1302&dpr=1&mstk=AUtExfBMIfQi4Kh4vMmyC73PxAA_VEz7YE3c_7lc5ktvZh2MaNiKP5BgMUAtoP3o1OQb5sFz1WH_TtFBhakR379AausR9bfbNn_AubAGP7Z8IhrCzu_acjFAeKTsJi4GWx3J0tboWs5EeojILG49bPylfgSnvRICHbRZVWABkd9s3jUlEEN9FpaRIT8Fu0L8BT1bifSsvQ-d8MEvnt6p1am7OSDR70XNfLphGgPz65jKnqFeGM74fHOI7UJ48V8O78uGgSEl4Eek0VGaJaNrjiDHbWX_4n_iz8IAi-0KZcCZgaFWdzUQvrq11o73wsJsmXWVptDHP47LB8-9zby4BydAVWRQ6Zne8p9aD84fOrTBmjddK4tZShQnyh3pg9zaBy1tG3br8wcyuhjU&csuir=1&mtid=eY2oaejyJ5269u8Puqr3yAM&udm=50 
1. [x] что может находится в stream().map а что нет
2. [x] стрим и полиморфизм
3. [ ] почему подсчет количества идет в Long а не в Integer
4. [ ] у разных коллекций разные стримы? (Set, HashMap, TreeMap...)
5. [ ] существует ли общее правило в reversed в sort
6. [ ] отличие Collectors.toMap от Collectors.groupingBy
7. [ ] паралельный стрим, что в какой поток попадает
8. [x] задачи по стримам 
9. [x] ```flatMap``` попробуй со списком объектов, внутри каждого из которых есть список или массив
10. [ ] ```flatMap``` что может быть в качестве параметра, типизация и перевод одного типа в другой
11. [x] Collectors

https://metanit.com/java/tutorial/10.1.php 

>[!question]- BASE
>```
>1. В основе лежат промежуточные операции и терминальные (после которых промежуточные не могут быть вызваны, так же как терминальные не могут быть вызваны)
>2. Промежуточных операций может быть несколько - терминальная операция одна Использует отложение выполнение лямбда выражений - то есть при вызове терминальной операции 
 >3. Во всех коллекциях начиная с jdk 8 есть метод stream через который можно получать объект Stream<T> , так же его можно получить через Arrays.stream(T[] array), Stream.of("Nikitin", "Bin") , InStream.of() , LongStream.of(), DoubleStream.of() ```
 
>[!question]- Отличие стрим потока от списка элементов
>- **Список (`List`)** — это структура данных. Он **хранит** элементы в памяти прямо сейчас. 
>- **Стрим (`Stream`)** — это набор инструкций. Он **не хранит** элементы, а лишь «прокачивает» их через себя от источника к результату. 
>- Стрим — ленив. Если вы написали `.map()`, Java не будет ничего делать, пока вы не вызовете `.collect()`. Элементы обрабатываются по одному только в момент необходимости.
>- Список не может быть бесконечным 
>- Стрим может быть бесконечным (например, стрим случайных чисел), так как он генерирует элементы «на лету».

>[!question]- `toMap` —
> collect.(Collectors.toMap()) - преобразует стрим в ассоциатвную коллекцию - ключ - значение , Может иметь от 2 до 4 параметров, третий парметр (в случае 4 параметров) - лмбда обработки дублей значений при одном ключе

>[!question]- `groupingBy` —
> collect.(Collectors.groupingBy()) - преобразует стрим в мапу - ключ - список значений 

>[!question]- `map` —
> это то, **как** мы меняем каждый элемент

>[!question]- `Collectors` — 
>это то, **во что** мы превращаем весь поток в самом конце.

>[!question]- `flatMap` — 
>разварачивает только один уровень вложенности, сколько уровней стлько раз надо вызвать этот метод - **нужен если нам надо получить у объекта не список , а каждый элемент списка**

>[!question]- сделать из списка ассоциативную коллекцию 
>Не убирая дубликаты
>4. 
>```
>Map<Integer, Fighter> mapObjById = list.stream().collect(Collectors.toMap(Fighter::getId, o -> o));
>```
>```
>Map<Integer, Fighter> mapObjById = list.stream().collect(Collectors.toMap(Fighter::getId,  Function.identity()));
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
>пиши 

>[!question]- Отображение это 
>Промежуточная операция map

>[!question]- Плоское Отображение это 
>Промежуточная операция flatMap
>Пример
>```
>Stream phoneStream = Stream.of(new Phone("iPhone 1 s", 28555), new Phone("iPhone 1 s", 28555));
>phoneStream.stream().flatMap(p->Stream.of(
>      String.format(p.getName(), ", ", p.getPrice()),
>      String.formate(p.getName(), ", discount; ", p.getPrice() / 2)
>    )
>).forEach(s->System.out.println(s));
>```
>Еще пример 
>```
>public List<FighterModel> getSortedFighters(List<FighterModel> fighterSortData) {
>    return Stream.of(BidStatus.APPROVED, BidStatus.WAIT_APPROVED, BidStatus.REJECT)
>        .flatMap(bs -> getFighterSortedList(fighterSortData, bs).stream())  .toList();
>    }
>```

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



docker exec -it -u root f28 sh