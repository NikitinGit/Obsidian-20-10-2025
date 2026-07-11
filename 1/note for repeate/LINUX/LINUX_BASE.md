# Base
1. [ ] аналоги исполняемых файлов .exe 
2. [ ] дискриптор процесса 
3. [ ] ss -ltnp | grep 8081 - как работает (netstat)

# [[CURL]]

# линукс 
За что отвечают системные директории в линукс
Базовые команды для работы с файлами
Stdin stdout steerr
Что такое пакеты, репозитории в линукс
Отличия натива, snap, flatpak (если хочется)
Что такое x и wayland (для общего развития)
Посмотреть разные дистрибутивы и их различия кратко



<font size="5">Типы программирования - ФП , императивное, реактивное ....</font>
<font size="3">Это текст с увеличенным шрифтом</font>

>[!question]- узнать данные о диструбутиве линукс и о сиситеме
>  ```
>  uname -a
>  ```
>   выводит краткую сводку: имя ядра, версию, архитектуру процессора и тип операционной системы.
>   ```
>   hostnamectl
>   ```
>   показывает детальную информацию о хосте, включая название дистрибутива, версию ядра, архитектуру и идентификатор шасси.
>   ```
>   lscpu
>   ```
>   детальная информация о процессоре (модель, количество ядер, потоков, кэш).
>   ```
>   free -h
>   ```
>   показывает объем оперативной памяти (всего, занято, свободно) в удобном для чтения формате.
>   

>[!question]-  Просмотр содержимого найденного файла
>cat /путь/к/файлу
>nano /путь/к/файлу

>[!question]- LINUX  это
>Linux — это семейство открытых операционных систем, подобных Unix, основанных на ядре Linux, которое впервые было выпущено Линусом Торвальдсом в 1991 году. Обычно оно распространяется в виде дистрибутива Linux (distro), который включает ядро вместе с системным программным обеспечением и библиотеками, создавая полноценную операционную систему. Linux широко используется на серверах, суперкомпьютерах, настольных компьютерах, ноутбуках и встроенных системах.

>[!question]- характеристики Unix-подобных систем
>- Многозадачность и многопользовательский режим;
>- Модульное ядро, управление памятью и устройствами;
>-  Иерархическую файловую систему с корневым каталогом; 
>- Командную оболочку (shell) для взаимодействия с системой;
>- Набор стандартных команд для работы с файлами, процессами и пользователями.

>[!question]- grep это 
>команда для поиска текста , пишется после пайпа  '\'
>Пример поиска в файле  заданного текста 
>```
>grep 'password' Documents/android\ studio\ jetbrains.txt 
>```
>вывести первые и последние н записей совпадений 
>```
>grep 'password' Documents/android\ studio\ jetbrains.txt | head -n 2
>grep 'password' Documents/android\ studio\ jetbrains.txt | tail -n 2
>```

>[!question]- найти слово во всех файлах папки (рекурсивно) — grep -rn
>```
>grep -rn "PING" /путь/к/папке
>```
> `-r` — recursive, рекурсивно по всем файлам в папке и подпапках
> `-n` — number, показывать номер строки совпадения
> Полезные варианты:
>```
># регистронезависимо (найдёт PING, ping, Ping)
>grep -rni "PING" /путь/к/папке
>```
>```
># только список файлов, без самих строк (-l = list)
>grep -rln "PING" /путь/к/папке
>```
>```
># искать только в нужных файлах (исключить минифицированные dist/ и т.п.)
>grep -rn --include="*.js" "PING" /путь/к/папке/src
>```
>```
># подсветить совпадения цветом (auto = только при выводе в терминал)
>grep -rn --color=auto "PING" /путь/к/папке
>```
> `--color=auto` — подсвечивает само найденное слово в выводе. Значения: `auto` (подсветка только в терминале, при записи в файл/pipe выключается — удобно), `always` (всегда, даже в pipe — цветовые коды попадут в файл), `never` (никогда). Часто `grep` уже настроен через alias `grep='grep --color=auto'`, тогда цвет работает по умолчанию.
>```
># посчитать количество совпадений (-c = count) по каждому файлу
>grep -rc "PING" /путь/к/папке
>```
> `-c` — count, вместо самих строк выводит ЧИСЛО совпадающих строк. При рекурсии (`-r`) показывает по каждому файлу: `файл:число` (файлы с 0 совпадений тоже попадают в список).
>```
># одно общее число совпадений во всём (без разбивки по файлам)
>grep -rh "PING" /путь/к/папке | wc -l
>```
> Нюанс: `-c` считает СТРОКИ с совпадением, а не сами совпадения — если в одной строке слово встречается 2 раза, это всё равно 1.
> Осторожно: без `--include` grep попадёт и в минифицированные файлы (весь код в одну строку) — тогда совпадение выведет огромную "простыню". Ограничивай папку (`/src`) или расширение.

>[!question]- ripgrep (rg) это 
>ищет рекурсивно строку в текущем катологе во всех файлах 
>```
>rg "error"
>```
>Пример поиска только в файлах с расширением .log:
>```
>rg --type=log "error"
>```

>[!question]- аналоги исполняемых файлов .exe
> 

>[!question]- Интерактивный режим это 
>Пиши

>[!question]- узнать где установлено приложение, мавен или сервис
>›which mvn
mvn not found
 ~ 
›which java
/etc/profiles/per-user/igor/bin/java
 ~ 
›which mysql    
/run/current-system/sw/bin/mysql

>[!question]- Стать супер пользователем (root)
>sudo su (окружение смешанное)
>sudo - super user DO
>su (swith user) - переключает юзера на root, запускает **новую оболочку (shell)** с правами root , вернуться обратно - exit. 
>'sudo su -' = чистое окружение 
>'sudo -i' = чистое окружение

>[!question]- Окружение (environment) это
>набор переменных, которые влияют на поведение программ
>echo $USER - имя юзера
>echo $HOME - ДОМАШНЯЯ ДИРЕКТОРИЯ
>Окружение может быть смешанным - 

>[!question]- как правильно удалить проект 
>rm -rf folder

>[!question]- запустить идею 
> cd jetbrains-toolbox-2.8.1.52155/bin/
>./jetbrains-toolbox

>[!question]- запустить исполняемый файл
>Узнать какой файл исполняемый = ls -l для всей директории или  ls -l jetbrains-toolbox для файла (если покажет переде ним -rwxr-xr-x то исполняемый)
>Как сделать файл исполняемым 
>Если файла нет в `ls -l` с правами `x`, нужно дать разрешение: chmod +x my_app
>Запустить ./my_app 
>в Linux **`.` (точка)** и **`./`** означают текущую директорию.  
  Просто `my_app` не сработает, если путь не прописан в `$PATH`.

>[!question]- посмотреть инфо о диструбутиве линукс на wsl
>cat /etc/os-release 
>lsb_release -a
>версия ядра uname -a


>[!question]- версия wsl 
>wsl --version

 >[!question]- перенести файл
 mv /home/igor/Pictures/Screenshots/ANginx.png /home/igor/Music/2/
>mv ~/Pictures/Screenshots/ANginx.png ~/Music/1

 > [!question]- через tree
> tree / -d | grep ".obsidian"

 > [!question]- создать папку
 >  mkdir /home/igor/Music 
 
 > [!question]- создать несколько папок 
 >  mkdir /home/igor/Music/{2,3,4}

 > [!question]- перенести файл
 >  mv /home/igor/Pictures/Screenshots/ANginx.png /home/igor/Music/2/
>   mv ~/Pictures/Screenshots/ANginx.png ~/Music/1

  > [!question]- перенести несколько файлов 
  > mv ~/Pictures/Screenshots/{bn.png,nginxN2.png} ~/Music/2

 > [!question]- копировать файл 
 > cp ~/Pictures/Screenshots/1.png ~/Music/3

 > [!question]- копировать несколько файлов 
 > cp ~/Pictures/Screenshots/{1.png,2.png,3.png} ~/Music/4

>[!question]- при копирование спрашивать подтверждение если файл существет
>cp -i ~/Pictures/Screenshots/1.png ~/Music/3

>[!question]- Очистить терминал
>Ctrl+L
>reset

>[!question]- Удалить директорию 
>rm -rf strikerstat
>rm -rf ~/.m2/repository/org/mapstruct/

# Симлинк

>[!question]- симлинк (symbolic link, символическая ссылка) — что это
>Специальный файл-**указатель** на другой путь (как ярлык в Windows). Открываешь ссылку → ОС прозрачно ведёт на файл-цель. Сам ссылочный файл данных не содержит.
>```
>ls -l /etc/nginx/sites-enabled/front.conf
># lrwxrwxrwx ... front.conf -> /etc/nginx/sites-available/front.conf
>```
>- **`l`** в начале прав (`lrwxrwxrwx`) = это link; **`->`** показывает, на что он ссылается (цель).
>- Правишь/бэкапишь **реальный файл-цель**, а не саму ссылку.
>- `grep -r` по умолчанию **не разворачивает** симлинки при рекурсии — поэтому поиск по папке с симлинками может «не найти» файл, хотя он там «есть».
>- Зачем нужны: напр. nginx держит ВСЕ конфиги в `sites-available/`, а «включённые» — это симлинки в `sites-enabled/`. Выключить сайт = удалить симлинк (`rm sites-enabled/xxx`), сам конфиг в `sites-available/` остаётся. Удобно включать/выключать, не теряя файлы.
>
>Команды:
>```
>ln -s /путь/цель /путь/ссылка   # создать симлинк
>readlink -f /путь/ссылка        # куда реально ведёт (развернуть до конца)
>cp -L источник назначение       # копировать сам файл-цель, а не ссылку
>rm /путь/ссылка                 # удалить ТОЛЬКО ссылку (файл-цель цел)
>```
>Ссылки на этот раздел: [[NGINX]] (sites-enabled → sites-available).
 
  > [!question]- Как найти папку на компьютере?  
> **Ответ:**  
> find / -type d -name ".obsidian" 2>dev/null 
> "2>dev/null" скрыть ошибки типа Permission dined

>[!question]- найти все файлы с расширением .docx
>find . -type f -name "*.docx"

1. Поиск файла по имени быстрый если примерно известна папка нахождения 
   sudo grep -rliw "nikitin" /var/www/html/ 2>/dev/null ^find
2. стоит попробовать быстрый поиск по всему серверу sudo grep -rliP --exclude-dir={proc,sys,dev,tmp,run,var/cache} "Hi\s+Nikitin" / 2>/dev/null 
3. Загрузить файл на сервер - 
   scp /home/igor/Pictures/Screenshots/ANginx.png root@109.71.242.134:/home/root/images  (можно использовать псевдоним)
4. создать папку mkdir images/nikotin 
5. перенести файл в папку, с которой работает вебсервер (например nginx) mv /home/root/images/ANginx.png /var/www/html/images/ - после этого файл будет открываться по http://boxcompetition.ru/images/ANginx.png ^media
6. ssh продлить время бездействия 
   сервер - sudo nano /etc/ssh/sshd_config 
      ClientAliveInterval 300      # Проверять активность каждые 300 секунд (5 минут)
     ClientAliveCountMax 3        # Максимум 3 проверки перед отключением (итого 15 минут)
    клиент -  nano ~/.ssh/config
        добавить 
        Host your-server.com
         ServerAliveInterval 60    # Отправлять "keepalive" каждые 60 секунд
         ServerAliveCountMax 5     # Максимум 5 попыток (итого 5 минут)
        или дудосить 
        while true; do echo -n ''; sleep 60; done  ^keepconnect
        
        
        новый файл на клиенте 
          # Глобальные настройки для ВСЕХ хостов (опционально)
Host *
  ServerAliveInterval 120     # Проверка активности каждые 2 минуты
  ServerAliveCountMax 3       # Разорвать соединение после 3 пропущенных проверок (итого 6 минут)
  TCPKeepAlive yes            # Включить TCP-keepalive
  IdentitiesOnly yes          # Использовать ТОЛЬКО указанные ключи

#Настройки для Bitbucket (оставлены как у вас)
Host bitbucket.org
  HostName bitbucket.org
  User git                    # Обязательно для Git
  AddKeysToAgent yes
  IdentityFile ~/.ssh/sshigor # Ваш ключ для Bitbucket
  IdentitiesOnly yes

#Настройки для вашего сервера по IP
Host myvps                    # Алиас для удобства (можно вызвать просто `ssh myvps`)
  HostName 109.71.242.134     # IP сервера
  User root                   # Логин (root)
  # IdentityFile ~/.ssh/sshigor # Раскомментируйте, если используете ключ
  # Port 2222                 # Раскомментируйте, если сменили порт SSH
        оригиналдьный файл 
```
#bitbucket
Host bitbucket.org
  AddKeysToAgent yes
  IdentityFile ~/.ssh/sshigor
```
7. Изменить пут ьхранения картинок с помощью nginx - добавить в /etc/nginx/sites-available/default 
   `location /images/ {         alias /home/root/images/;        autoindex on;          # опционально, чтобы видеть список файлов в браузере        try_files $uri =404;   # отдавать 404, если файл не найден    }`
   перезагрузить nginx 
   sudo systemctl reload nginx 
   
8. 
   




Чтобы найти файл **с неизвестным именем**, содержащий текст **`Hi Nikitin`** (без учёта регистра), используйте следующие команды:

---

### **1. Поиск во всех файлах сервера (медленно, но надёжно)**

bash

sudo grep -rli "hi nikitin" / 2>/dev/null

#### **Разбор команды:**

- `grep` — поиск текста.
    
- `-r` — рекурсивно (включая подпапки).
    
- `-l` — выводить только имена файлов (без самого текста).
    
- `-i` — игнорировать регистр (`Hi Nikitin` = `hi nikitin`).
    
- `"hi nikitin"` — искомый текст.
    
- `/` — поиск от корня сервера.
    
- `2>/dev/null` — скрыть ошибки "Permission denied".
    

**Пример вывода:**

/var/www/site/index.html
/home/user/project/unknown_file.txt
/etc/config/some_config.conf

---

### **2. Ускоренный поиск (только в текстовых файлах)**

Если искать только в файлах с текстовым содержимым (исключая бинарные файлы):

bash

sudo find / -type f -exec grep -l -i "hi nikitin" {} + 2>/dev/null

---

### **3. Поиск в конкретных директориях**

Если догадываетесь, где может лежать файл (например, в `/var/www/` или `/home/`):

bash

sudo grep -rli "hi nikitin" /var/www/ /home/ 2>/dev/null

---

### **5. Если файлов слишком много**

Можно ограничить поиск по расширению (например, только `.html`, `.txt`, `.conf`):

bash

sudo grep -rli --include="*.{html,txt,conf}" "hi nikitin" / 2>/dev/null

---

### **Пример работы**

**Шаг 1:** Поиск

bash

sudo grep -rli "hi nikitin" / 2>/dev/null

**Вывод:**

/var/www/site/index.html
/etc/nginx/config.conf

**Шаг 2:** Проверка

bash

cat /etc/nginx/config.conf

**Содержимое:**

# Конфиг для Nikitin
server {
    listen 80;
    server_name hi_nikitin.example.com;
}

---
### 2. **Поиск с учётом пробелов и форматирования**

Если текст записан с нестандартными пробелами или переносами:

bash

sudo grep -rliP "Hi\s+Nikitin" /var/www/html/ 2>/dev/null

- `-P` — включает **Perl-совместимые регулярные выражения**.
    
- `\s+` — ищет 1 или более пробельных символов (обычный пробел, табуляция, перенос строки).
    

---

### 3. **Поиск в файлах с нестандартными кодировками**

Если файл сохранён в кодировке **UTF-16** или другой:

bash

sudo find /var/www/html/ -type f -exec file {} + | grep "text"

Затем для найденных текстовых файлов:

bash

sudo iconv -f UTF-16 -t UTF-8 /var/www/html/file.txt | grep -i "nikitin"

---

### 4. **Проверка на скрытые символы или HTML-теги**

Если текст внутри HTML/CSS/JS:

bash

sudo grep -rli "<[^>]*>Nikitin" /var/www/html/ 2>/dev/null

Ищет в тегах (`<p>Hi Nikitin</p>`).

---

### 5. **Альтернативные инструменты**

#### Ripgrep (быстрее `grep`):

bash

sudo rg -i "hi nikitin" /var/www/html/

Установка: `sudo apt install ripgrep`.

#### Silver Searcher (`ag`):

bash

sudo ag -i "hi nikitin" /var/www/html/

Установка: `sudo apt install silversearcher-ag`.

--- 
### **Важно!**

- Поиск по всему серверу (`/`) может занять **много времени** (особенно на больших дисках).
    
- Если знаете примерное расположение файла, укажите конкретную папку (например, `/var/www/`).
    
- Для сложных случаев можно использовать `ag` (`the_silver_searcher`) или `ripgrep` (`rg`), но их нужно предварительно установить.
    

---
