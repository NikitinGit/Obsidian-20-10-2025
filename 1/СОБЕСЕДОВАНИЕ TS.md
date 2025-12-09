
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

