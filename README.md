# Деплой сайта на VPS

- Используем Docker контейнер для веб приложения чтобы ограничить доступ извне Docker с помощью network
- Nginx находится в 1 контейнере обслуживающем все веб приложения.

# Quickstart

1) Поднимаем nginx контейнер с пустым конфигом
2) Создаём сеть для общения nginx с веб приложениями

        docker network create nginx-network
3) Добавляем nginx контейнер в сеть

        docker network connect nginx-network nginx_container
4) Поднимаем web_app контейнер без открытых портов
5) Добавляем web_app в сеть

       docker network connect nginx-network nginx
6) Настраиваем конфиг nginx для web_app
7) Перезапускаем nginx

        docker restart nginx
8) Скачиваем скрипт установки сертификата 

        curl -L https://raw.githubusercontent.com/wmnnd/nginx-certbot/master/init-letsencrypt.sh > init-letsencrypt.sh
9) Предварительно настраиваем скрипт под web_app ( **staging = 1** )

10) Устанавливаем разрешение на запуск для скрипта

         chmod +x init-letsencrypt.sh
11) Запускаем скрипт

        sudo ./init-letsencrypt.sh
12) Если все хоккей, то редактируем скрипт ( **staging = 0** ) и заного запускаем




## Additions

#### Откат локальных изменений Git
    git reset --hard HEAD

#### Create image
    docker build . --tag image_name

#### Create container 
    docker run -d --name container_name image_name 

#### Run docker-compose
    docker-compose up --build -d

## КОНФИГ NGINX
    
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



