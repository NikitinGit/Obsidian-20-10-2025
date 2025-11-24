# NGINX
установка сертифката 
https://sky.pro/wiki/html/kak-nastroit-https-na-vashem-sajte/ 

>[!question]- каак сделать так чтобы он запускаался после ребутва автоматически 
> sudo systemctl enable nginx.service



# To do 
1. **Поиск по части имени**

Если название содержит слово "parking" (например, `my_parking_file.txt`):
sudo find / -type f -name "*parking*" 2>/dev/null


### **1. `user www-data;`**

- **Что делает**: Задаёт пользователя и группу, от имени которых будут работать **worker-процессы Nginx**.
    
- **Почему `www-data`**:
    
    - В Ubuntu/Debian пользователь `www-data` создаётся специально для веб-серверов (Nginx/Apache).
        
    - Это повышает безопасность: процессы не запускаются от `root`.
        

---

### **2. `worker_processes auto;`**

- **Что делает**: Определяет количество **рабочих процессов** Nginx.
    
- **`auto`**: Nginx автоматически выбирает оптимальное число (обычно равно количеству CPU-ядер).
    
- **Можно указать вручную**:
    
    nginx
    
    Copy
    
    worker_processes 4;  # Например, для 4-ядерного CPU.
    

---

### **3. `pid /run/nginx.pid;`**

- **Что делает**: Указывает путь к файлу, где хранится **PID (Process ID)** главного процесса Nginx.
    
- **Зачем нужно**:
    
    - Система и скрипты используют этот файл для управления Nginx (перезагрузка, остановка).
        
    - Пример:
        
        bash
        
        Copy
        
        sudo kill -HUP $(cat /run/nginx.pid)  # Перезагрузка конфига.
        

---

### **4. `error_log /var/log/nginx/error.log;`**

- **Что делает**: Задаёт путь к файлу **логов ошибок**.
    
- **Уровни логирования**:  
    Можно добавить уровень серьезности (например, `error_log /var/log/nginx/error.log warn;`).  
    Доступные уровни: `debug`, `info`, `notice`, `warn`, `error`, `crit`, `alert`, `emerg`.
    

---

### **5. `include /etc/nginx/modules-enabled/*.conf;`**

- **Что делает**: Подключает все конфигурационные файлы из папки `/etc/nginx/modules-enabled/`.
    
- **Зачем**:
    
    - В этой папке обычно лежат симлинки на модули Nginx (например, `ngx_http_geoip_module.conf`).
        
    - Позволяет включать/отключать модули без правки основного `nginx.conf`.





на  сервере timeweb с ОС ubuntu  - запущен проект на vue/nuxt как узнать где он находится и как с ним взаимодействует nginx 


ls -la /var/www/strikerstat-front/pages/ 
 