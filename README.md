#Деплой сайта на VPS

- Используем Docker контейнер для веб приложения чтобы ограничить доступ извне Docker с помощью EXPOSE
- Nginx находится только в 1 общем контейнере для всех веб приложений.


1) Поднимается Контейнер с Nginx

Добавление нового WebApp
2) Поднимается Контейнер с WebApp (* с certbot если нужен ssl)
3) Изменяется конфиг Nginx
4) Перезапуск Nginx
5) * установка init-letsencrypt.sh
6) * изменение конфига init-letsencrypt.sh
7) * запуск init-letsencrypt.sh ($ sudo ./init-letsencrypt.sh)

###УСТАНОВКА init-letsencrypt.sh

    curl -L https://raw.githubusercontent.com/wmnnd/nginx-certbot/master/init-letsencrypt.sh > init-letsencrypt.sh


###КОНФИГ NGINX
    
    server {
        listen 80;  # слушаем порт 80
    
        server_name labao.ru www.labao.ru;  # обрабатываем запросы только на эти домены
    
        location /static/ {
            root /var/html/;
        }
    
        location /media/ {
            root /var/html/;
        }
    
    
        location / {
            return 301 https://$host$request_uri;  # редирект на https
        }
    
        location /.well-known/acme-challenge/ {  # для установки ssl certbotом
            root /var/www/certbot;
        }
    }
    
    server {
        listen 443 ssl;
        server_name labao.ru www.labao.ru;
    
        ssl_certificate /etc/letsencrypt/live/labao.ru/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/labao.ru/privkey.pem;
    
        location / {
            proxy_pass http://web_app:80;  # редирект на webapp с контейнером
        }
    }
    
    server {
        listen 80;
        listen [::]:80;  # запись для ipv6
    
        server_name 5.188.50.249;
    
        location / {
            proxy_pass http://web_app:80;
        }
    }
    
    server {
        listen 80 default_server;  # default_server - если никакое server_name не подошло будет обрабатываться здесь
        listen [::]:80;
    
        server_name _;  # для всех имен
    
        location / {
            return 404;
        }
    }



