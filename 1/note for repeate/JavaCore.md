
>[!question]- лямбда в методе forEach, что с его парамметром
>```
>bidFighterList.forEach(bidFighter -> {
>bidFighter.setApproved(newStatus.num);
>bidFighter.setApproved(BidStatus.APPROVED.num);  
});
>```

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

