# Корутины — асинхронность в Kotlin

Корутины — лёгкие "потоки" (легче чем `Thread`), управляются библиотекой, а не ОС. Один JVM-поток может выполнять тысячи корутин.

---

>[!question]- что такое suspend-функция
>Функция которая может **приостанавливаться** без блокирования потока.
>```kotlin
>suspend fun loadUser(id: Int): User {
>    val data = httpClient.get("/users/$id")    // suspend — не блокирует поток
>    return parse(data)
>}
>```
>Можно вызывать **только** из другой suspend-функции или корутины.
>Под капотом — CPS-трансформация: компилятор превращает функцию в state machine с continuation.

>[!question]- запуск корутины — launch / async
>```kotlin
>// launch — fire-and-forget, возвращает Job
>val job = scope.launch {
>    loadUser(1)
>}
>job.join()                     // ждём завершения
>job.cancel()                   // отменяем
>
>// async — возвращает Deferred<T>, можно получить результат
>val deferred = scope.async {
>    loadUser(1)
>}
>val user = deferred.await()
>```

>[!question]- CoroutineScope
>Контекст жизненного цикла корутин. Когда scope отменён — все его дочерние корутины отменяются.
>```kotlin
>val scope = CoroutineScope(Dispatchers.IO + SupervisorJob())
>scope.launch { ... }
>scope.cancel()                  // отменит все вложенные
>```
>**Готовые scope**:
>- `GlobalScope` — на весь процесс, **избегать** — нет structured concurrency.
>- `viewModelScope` / `lifecycleScope` — Android.
>- `runBlocking { }` — для main/test, блокирует текущий поток.

>[!question]- Dispatchers
>Куда отправлять выполнение:
>- `Dispatchers.Default` — CPU-bound (N потоков = N ядер).
>- `Dispatchers.IO` — IO-bound (большой пул, ~64+).
>- `Dispatchers.Main` — UI поток (Android, JavaFX, Swing).
>- `Dispatchers.Unconfined` — где попало, осторожно.
>
>```kotlin
>launch(Dispatchers.IO) { db.query() }
>withContext(Dispatchers.Default) { heavyCompute() }
>```

>[!question]- structured concurrency
>Главный принцип: **дочерние корутины не могут пережить родителя**.
>```kotlin
>coroutineScope {                    // suspend-функция
>    launch { taskA() }
>    launch { taskB() }
>}                                   // ждёт обеих, исключения проброшены
>```
>Если одна упала — отменяются все остальные.

>[!question]- параллельное выполнение
>```kotlin
>coroutineScope {
>    val a = async { loadUserA() }
>    val b = async { loadUserB() }
>    a.await() + b.await()
>}
>
>// или
>val results = items.map { async { process(it) } }.awaitAll()
>```

>[!question]- отмена корутины
>Кооперативная — корутина должна сама проверять `isActive` или использовать suspend-функции (они проверяют отмену).
>```kotlin
>launch {
>    while (isActive) { ... }
>    // или delay(100) — сам бросит CancellationException
>}
>```
>`CancellationException` — не баг, его нельзя проглатывать в `catch (e: Exception)`. Используй `catch (e: Exception)` + `if (e is CancellationException) throw e`.

>[!question]- exception handling
>```kotlin
>val handler = CoroutineExceptionHandler { _, e -> println("error $e") }
>scope.launch(handler) { ... }
>
>// или try/catch внутри
>launch {
>    try { riskyOp() } catch (e: Exception) { ... }
>}
>```
>**В `async`** — исключение бросается из `await()`, не сразу.

>[!question]- SupervisorJob
>В обычном `Job` — падение одной дочерней отменяет всех.
>`SupervisorJob` — падение одной не влияет на других.
>```kotlin
>val scope = CoroutineScope(SupervisorJob())
>// или
>supervisorScope { ... }
>```

>[!question]- withContext
>Переключает диспетчер без запуска новой корутины:
>```kotlin
>suspend fun loadData() = withContext(Dispatchers.IO) {
>    db.query()
>}
>```

---

# Flow — асинхронные потоки данных

Аналог `Sequence` но асинхронный + back-pressure.

>[!question]- Flow base
>```kotlin
>fun numbers(): Flow<Int> = flow {
>    repeat(5) {
>        delay(100)               // suspend OK
>        emit(it)
>    }
>}
>
>numbers().collect { println(it) }
>```

>[!question]- операторы Flow
>```kotlin
>flow.map { it * 2 }
>flow.filter { it > 0 }
>flow.flatMapConcat { ... }       // sequential
>flow.flatMapMerge { ... }        // parallel
>flow.flatMapLatest { ... }       // отмена предыдущей
>flow.debounce(100)
>flow.distinctUntilChanged()
>flow.flowOn(Dispatchers.IO)      // upstream диспетчер
>```

>[!question]- StateFlow / SharedFlow
>**StateFlow** — hot flow с текущим значением (как `Subject` в RX):
>```kotlin
>val state = MutableStateFlow(0)
>state.value = 1
>state.collect { ... }
>```
>**SharedFlow** — hot flow без обязательного значения, multicast.

---

# Структуры синхронизации

>[!question]- Mutex
>```kotlin
>val mutex = Mutex()
>mutex.withLock { sharedState++ }
>```
>Не блокирует поток (в отличие от `synchronized`), а приостанавливает корутину.

>[!question]- Channel
>FIFO-канал для общения между корутинами:
>```kotlin
>val ch = Channel<Int>()
>launch { ch.send(1); ch.send(2); ch.close() }
>for (x in ch) println(x)
>```

---

# Корутины vs Threads

| | Threads | Корутины |
|---|---|---|
| Стоимость | ~1MB стек | ~КБ контекст |
| Сколько можно | сотни | сотни тысяч |
| Управление | ОС | библиотека |
| Блокировка | `Thread.sleep` | `delay` (не блокирует поток) |
| Отмена | `interrupt()` (не везде работает) | `cancel()` (кооперативная) |

---

# Шпаргалка

- `suspend fun` можно вызывать только из корутины или другой suspend-функции.
- `launch` — без результата, `async`/`await` — с результатом.
- `coroutineScope` — структурированная concurrency, ждёт всех детей.
- `Dispatchers.IO` для блокирующего IO, `Default` для CPU-bound.
- `Flow` — асинхронный `Sequence` с back-pressure.
- `Mutex.withLock` вместо `synchronized` для корутин.
- `CancellationException` не глотать в `catch (Exception)`.
- `GlobalScope` — anti-pattern, используй явный scope с lifecycle.
