
1. [ ] ...createBattleMethods('/api/main-judge/battles') изучи в своем проекте

# css 
>[!question]- выровнять кнопку по центру
>justify-center в тайлвинде -  следует после контейнера flex 

# ts

>[!question]- ?? это 
> Оператор ?? (Nullish Coalescing) 
> пример 
> schedulingEnabled: eventToEdit.value.schedulingEnabled ?? false
>  ?? - это оператор нулевого слияния (nullish coalescing operator). Он работает так:
>   Если левая часть (eventToEdit.value.schedulingEnabled) равна null или undefined, используется правая часть (false) 
>    Если левая часть имеет любое другое значение (включая true, false, 0, ""), используется левая часть 

>[!question]- Вопросительный знак ? в TypeScript
> schedulingEnabled?: boolean
> Знак ? означает, что это необязательное поле (optional property). Это значит:
> - Объект может содержать это поле: { schedulingEnabled: true }
> - Или может не содержать: { } - и это тоже валидно
> - Зачем это нужно:
> - Для обратной совместимости! Старые данные в базе не содержат поле schedulingEnabled, и когда API возвращает эти
> - данные, TypeScript не будет ругаться на отсутствие этого поля.


>[!question]- TS интерфейсы 
>пиши

>[!question]- TS как объявить переменные  
>пиши

>[!question]- TS создать класс
>пиши

>[!question]- TS создать объект
>пиши

>[!question]- TS конструктор
>пиши

>[!question]- TS создать метод
>пиши

>[!question]- Проверка типов: `nuxi typecheck` (vue-tsc) vs `tsc`
> **Команда (Nuxt/Vue проект):**
> ```bash
> cd .../frontend && npx nuxi typecheck > /tmp/tc.log 2>&1; echo "EXIT=$?"; grep -iE "error TS" /tmp/tc.log | head -20
> ```
> Разбор:
> - `&&` — выполнить дальше только если `cd` успешен
> - `> /tmp/tc.log` — stdout в файл; `2>&1` — stderr туда же (всё в один лог)
> - `;` — выполнить следующее безусловно
> - `echo "EXIT=$?"` — код возврата прошлой команды; `EXIT=0` = типы ок
> - `grep -iE "error TS"` — вытащить из лога строки TS-ошибок (`error TS2322: ...`); `-i` без регистра, `-E` расширенный regex
> - `| head -20` — первые 20 строк
> - Зачем лог в файл: Nuxt сыплет много предупреждений в консоль, грепом достаём только важное
>
> **Vue-only или TS вообще?**
> `nuxi typecheck` под капотом запускает **`vue-tsc`**, не обычный `tsc`.
> - `tsc` — умеет только `.ts`/`.tsx`, **не понимает** `.vue`
> - `vue-tsc` = `tsc` + поддержка Vue SFC: проверяет типы и в `.ts`, **и** внутри `<script setup>`/`<template>` в `.vue`
> - Т.е. это НЕ «только Vue» — проверяется весь TS проекта, просто плюс поддержка `.vue`
> - Чистый TS-проект (без Vue): аналог — `npx tsc --noEmit` (`--noEmit` = только проверка, без генерации JS)

