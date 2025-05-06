1. Создать папку на сервере mkdir -p /home/root/myapp 
2. Выйти exit
3. Копируем клас файл на сервер - scp SimpleHttpServer*.class root@109.71.242.134:/home/root/myapp/  
4. Подключаемся по ssh и переходим в папку с файлом cd /home/root/myapp/
5.  Запуск java SimpleHttpServer
6. Запуск в фоне (демон) nohup java SimpleHttpServer > app.log 2>&1 &