1. Установка  sudo apt install nginx-core -y  # Базовая версия Или 
   sudo apt install nginx-extras -y  # С дополнительными модулями
2. Проверка утсановки nginx -v 
3. Проверка статуса запуска sudo systemctl status nginx
4. Файлы находятся в /etc/nginx
5. Проверка конифигурации nginx.conf на синтаксис sudo nginx -t
6. Перезапуск nginx - sudo systemctl restart nginx
7. Остановить сервис nginx - sudo systemctl stop nginx
8. Перенаправить запрос к фалам 
