
https://www.google.com/search?q=java+%D1%80%D0%B0%D0%B7%D0%BD%D0%B8%D1%86%D0%B0+%26%26+%D0%B8+%26&gs_lcrp=EgZjaHJvbWUyBggAEEUYOTIICAEQABgIGB4yCAgCEAAYCBgeMggIAxAAGAgYHjIICAQQABgIGB4yCAgFEAAYCBgeMggIBhAAGAgYHjIICAcQABgIGB4yBggIEEUYQTIICAkQABgIGB4yCAgKEAAYCBgeMggICxAAGAgYHjIICAwQABgIGB4yCAgNEAAYCBgeMggIDhAAGAgYHtIBCjI4ODI5ajBqMTaoAgGwAgE&sourceid=chrome-mobile&ie=UTF-8&fbs=ADc_l-YWHxjctacVdI3-Iqscc_X6ZBg4CBYeUg71Nw4kHqUpT3Ykwrvi8PHco0mjTplrZGmvuoJNWYi3Iq0yH4mR3dknjy9lyjGqwqdIoH3hAn02rYfRm-px5NOj80sjoeOuCw5mGks2GQGF_idyM9mE7m4r7C3izLw0z7OdvFocOiuS3ST_4f1K9aFCVDANTrTQQTAnwDmPycN5VZOSy7MqCkkX7AMRy5J_dMHoS6v6CpvdVGHhizdyUDzhZAWIKXLT-V1_a2O9eB633e0rO6YJoM-gPm5-Lg&aep=10&ntc=1&mstk=AUtExfAjm-fLnKn4ueg8ZIl8m9QfKfrOn5PLmh7HS0VD2bi0mVIzArcdDhgy2vVDdvaYu_RVKGpjeK7pOEqAOkti9v_jdiWHn_KFcS63ocXh1UQ-A105MkSYk0athuT9ed9w0etc3Ui5bJZCNyQyI29EpbU75hFa6buI3dlQ7vEnZ_mmukMDnEbtPEBvvLM_OH-PgNJhCfaSk0uY08Fb8wq3EAqSYT4XAj1NalswhX61bQjX-Fx9nJs9X7APD_XQxTIqjbGlgO1s6FWF9exg1q85MX5f103qyg1Xj_vUNlpDsjk4auh3bkZYc1v6tAqb9AGLYA8kxkefqAPMKw&csuir=1&aioh=3&mtid=uoOeabD8Ds6M9u8PjbPP-Qo&udm=50#lfId=ChxjMe

>[!question]- & это 
>«амперсанд» - побитовое И сумма всех  битов в двоичном предсталение числа -  если оба бита 1 - то в результате получается 1, иначе  0
>```
>0101
>1101
>=
>0101
>```

>[!question]- | это 
>пайп - побитовое ИЛИ 0 сумма всех  битов в двоичном предсталение числа -  если хотя бы один бит = 1 - то в результате получается 1, иначе  0
>```
>0101
>1101
>=
>1101
>```

>[!question]- ^ это 
> каретка - исключающее побитовое ИЛИ  (часто называют "контролем различий") -  сли биты разные- то в результате получается 1, иначе  0
>```
>0101
>1101
>=
>1000
>```

>[!question]- ~ это 
>тильда - Оператор **Bitwise Complement** (`~`) — это унарная операция, которая инвертирует каждый бит в двоичном представлении числа. Простыми словами: **0 превращается в 1, а 1 — в 0**.
>```
>0101
> "~"
>=
>1010
>```

>[!question]- '>>'
>сдвиг всех битов  на 1 вправо (деление на 2)
>```
>1010
>   ">>"
>=
>0101
>```
>если не четное то  - остаток отбрасывается (-1)

>[!question]- '<<' это 
>сдвиг всех битов  на 1 влево (умножение на 2)
>```
>0101
>   "<<"
>=
>1010
>```
>1. Скорость - процессору сдвинуть биты быстрее  чем  выполнить полноценое деление
>2. Упаковка данных - Можно запихнуть несколько маленьких чисел в одно большое - например упаковать 2 цвета (rgb)

>[!question]- '>>>' это 
> беззнаковый сдвиг - при сдвиге вправо всегда вставляется 0 слева игнрорируя знак числа - плюс минус - пример 
>```
> int a = 8; // В битах: 0000...00001000
int b = a >>> 2; 
// Результат: 2 (0000...00000010)
>```

>[!question]- где исползуется 
>  в rgb - & , | - в одном int сохраняются все 3 цвета (0 - 255) 
>```
> int red = 255;  
> int green = 128;
> int blue = 64;
> int rgb1 = (red << 16) | (green << 8) | blue;
> int green2 = (rgb1 >> 8) & 0xFF;
>```

>[!question]- правила инициализации 
>Используйте префикс **`0b`** для целых чисел. Для `boolean` и дробных чисел используйте стандартные значения, так как их битовая структура скрыта за правилами виртуальной машины Java. В Java есть несколько способов записи чисел: **двоичный** (binary), **шестнадцатеричный** (hex). Восьмеричная система в Java задается ведущим **нулем**. Она почти не используется в современном Spring/Enterprise, в отличие от **шестнадцатеричной (`0x`)**, которая повсеместна. 

