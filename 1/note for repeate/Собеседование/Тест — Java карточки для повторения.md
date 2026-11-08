# Тест: Java-карточки для повторения

#flashcards

Чем `Collections` отличается от `Collectors`?
?
`java.util.Collections` — утилиты для готовых коллекций: например, `sort()` и `shuffle()`.
<!--SR:!fsrs,2026-11-16T16:16:21.251Z,10,8.2956,1,2,1,0,0,2026-11-06T16:16:21.251Z,id=fu37t8-->

`java.util.stream.Collectors` — фабрики коллекторов для `Stream.collect()`: например, `groupingBy()`, `counting()` и `joining()`.

```java
Collections.shuffle(orders); // orders — изменяемый список

Map<Long, Long> counts = orders.stream()
    .collect(Collectors.groupingBy(
        Order::customerId,
        Collectors.counting()));
```

---

Stream API: как посчитать оплаченные заказы каждого клиента?

**Дано:** `record Order(long id, long customerId, boolean paid) {}`.

**Сигнатура:** `static Map<Long, Long> countPaidOrders(List<Order> orders)`.

Вход содержит оплаченные и неоплаченные заказы. Список и его элементы не равны `null`.

Верните число оплаченных заказов по каждому клиенту; клиентов без оплаченных заказов в карту не включайте.

**Пример входа:** `(1, 10, true)`, `(2, 10, false)`, `(3, 10, true)`, `(4, 20, true)`, `(5, 30, false)`.

**Ожидается:** `{10=2, 20=1}`. Пустой вход → пустая карта. Порядок ключей не важен.

Разрешены стандартные `java.util.*` и `java.util.stream.*` из Java 17.

Дополнительно: какие операции ленивые и где запускается обработка?
?
```java
static Map<Long, Long> countPaidOrders(List<Order> orders) {
    return orders.stream()
        .filter(Order::paid)
        .collect(Collectors.groupingBy(
            Order::customerId,
            Collectors.counting()));
}
```
<!--SR:!fsrs,2026-11-08T07:40:11.101Z,2,2.3065,2.11121424,2,2,0,0,2026-11-06T07:40:11.101Z,id=70qbxr-->

`filter()` — ленивая промежуточная операция. `collect()` — терминальная операция, которая запускает обработку. `Collectors.counting()` возвращает коллектор для подсчёта элементов в каждой группе; итоговый счётчик имеет тип `Long`.

## После теста

Запишите рядом с карточкой или сообщите результат:

- обе ли карточки найдены;
- был ли ответ скрыт;
- какие сроки показа предложили кнопки;
- сохранился ли прогресс после перезапуска Obsidian.
