1. Создать папку на сервере mkdir -p /home/root/myapp 
2. Выйти exit
3. Копируем клас файл на сервер - scp SimpleHttpServer*.class root@109.71.242.134:/home/root/myapp/  
4. Подключаемся по ssh и переходим в папку с файлом cd /home/root/myapp/
5.  Запуск java SimpleHttpServer
6. Запуск в фоне (демон) nohup java SimpleHttpServer > app.log 2>&1 &
7. Посмотреть логи приложения - перейти в папку с прилой **cd /home/root/myapp/** и команда **<span>cat app.log</span>  или tail -f app.log 
8. info about error - grep -i "error\|exception\|fatal" app.log 
9. посмотреть статус выполнения задаччи - echo $? 
10. остановить все процессы java
     pkill -f java 
   11. Полсмотреть нагрузку на сервер  - htop 