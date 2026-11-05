# Асинхронное программирование и CompletableFuture

## Главное различие

- **Многопоточность** — несколько потоков могут выполнять работу параллельно.
- **Асинхронность** — вызывающий код не блокируется в ожидании длительной операции и может продолжить работу.

Это не одно и то же: асинхронный код может работать в одном потоке, а многопоточный — ожидать результат и тем самым блокировать вызывающий поток.

## CompletableFuture

`CompletableFuture<T>` представляет результат, который появится позже.

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    return "Готово";
});
```

`supplyAsync` запускает задачу и возвращает `CompletableFuture` с её результатом. Без указанного `Executor` используется общий `ForkJoinPool.commonPool()`.

### Ожидание результата

```java
String result = future.join();
```

- `join()` ждёт завершения и при ошибке бросает unchecked `CompletionException`.
- `get()` ждёт завершения и требует обработки checked `ExecutionException` и `InterruptedException`.

В асинхронной цепочке обычно лучше не вызывать `join()` внутри промежуточных действий: вместо этого использовать `then...` методы.

## thenCombine: две независимые задачи

`thenCombine` объединяет результаты **двух уже запущенных независимых** Future.

```java
CompletableFuture<Long> userCount =
        CompletableFuture.supplyAsync(userRepository::count);

CompletableFuture<List<String>> projectNames =
        CompletableFuture.supplyAsync(() -> projectRepository.findAllWithOwner()
                .stream()
                .map(Project::getName)
                .toList());

CompletableFuture<CombineExampleDto> result =
        userCount.thenCombine(projectNames, CombineExampleDto::new);
```

Схема:

```text
userCount     ─┐
               ├─ thenCombine → общий результат
projectNames  ─┘
```

Операция-объединитель выполнится, только если обе Future завершились нормально. Один `thenCombine` объединяет две Future; для большего количества можно строить цепочку или использовать `CompletableFuture.allOf(...)`.

```java
first.thenCombine(second, Integer::sum)
     .thenCombine(third, Integer::sum);
```

### Ошибки в thenCombine

Если хотя бы одна Future завершается с ошибкой, общий результат тоже завершается с ошибкой и функция объединения не вызывается.

```java
try {
    userCount.thenCombine(projectNames, CombineExampleDto::new).join();
} catch (CompletionException ex) {
    Throwable originalError = ex.getCause();
}
```

Вторая задача не отменяется автоматически. Она продолжает выполняться; `thenCombine` не является fail-fast. Для обработки и успешного, и ошибочного сценария подходит `handle`:

```java
combined.handle((result, exception) -> {
    if (exception == null) {
        return "Успех: " + result;
    }

    Throwable cause = exception instanceof CompletionException && exception.getCause() != null
            ? exception.getCause()
            : exception;
    return "Ошибка: " + cause.getMessage();
});
```

Проверять нужно `exception == null`, а не `result != null`: успешная Future тоже может вернуть `null`.

## thenCompose: вторая задача зависит от первой

`thenCompose` нужен, когда результат первой Future нужен, чтобы **запустить вторую** Future.

```java
CompletableFuture<ComposeExampleDto> result = findProjectAsync(projectId)
        .thenCompose(project ->
                findTasksByProjectIdAsync(project.getId())
                        .thenApply(tasks -> new ComposeExampleDto(
                                project.getId(),
                                project.getName(),
                                tasks.stream().map(Task::getTitle).toList())));
```

Схема:

```text
найти проект → получить project.id → найти задачи проекта → общий результат
```

`thenCompose` похож на `flatMap` у `Optional` и `Stream`: он убирает вложенность.

Без него:

```java
CompletableFuture<CompletableFuture<List<Task>>>
```

С ним:

```java
CompletableFuture<List<Task>>
```

Если первая Future завершается с ошибкой, функция внутри `thenCompose` не вызывается, и вторая задача не запускается.

## thenApply, thenAccept и handle

> [!question]- Когда использовать `thenApply`?
> 
> Когда нужно синхронно преобразовать готовый результат Future. Возвращает новую Future с преобразованным значением.

> [!question]- Когда использовать `thenCompose`?
> 
> Когда результат первой Future нужен для запуска второй асинхронной операции, которая тоже возвращает Future. Убирает вложенность `CompletableFuture<CompletableFuture<T>>`.

> [!question]- Когда использовать `thenCombine`?
> 
> Когда есть две независимые Future и для следующего шага нужны результаты обеих. Возвращает новую Future с объединённым значением.

> [!question]- Когда использовать `thenAccept`?
> 
> Когда результат нужен только для побочного эффекта: логирования, вывода в консоль, сохранения. Возвращает `CompletableFuture<Void>`.

> [!question]- Когда использовать `handle`?
> 
> Когда нужно обработать и успешное выполнение, и ошибку. Обработчик получает `result` и `exception`, один из которых обычно равен `null`.

> [!question]- Когда использовать `exceptionally`?
> 
> Когда нужен запасной результат только при ошибке. При успехе исходный результат передаётся дальше без изменений.

## Примеры в NauJava

В ветке `completablefuture-examples` добавлены маршруты:

```text
GET /examples/completable-futures/combine
GET /examples/completable-futures/compose/{projectId}
GET /examples/completable-futures/combine-error
```

- `combine` считает пользователей и получает проекты параллельно;
- `compose` сначала находит проект, затем его задачи;
- `combine-error` показывает, что ошибка первой Future не отменяет вторую автоматически.

## Практические правила

1. Независимые операции — `thenCombine` или `allOf`.
2. Зависимые операции — `thenCompose`.
3. Не блокировать цепочку вызовом `join()` раньше времени.
4. Явно обрабатывать ошибки через `handle`, `exceptionally` или `whenComplete`.
5. Для production-кода лучше передавать собственный `Executor`, а не безусловно использовать общий `ForkJoinPool.commonPool()`.
