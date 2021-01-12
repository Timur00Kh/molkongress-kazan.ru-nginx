# molkongress-kazan.ru-nginx

Типо инструкция как настраивать SSL в nginx c помощью certbot на Ubuntu

## Nginx установка

```bash
sudo apt-add-repository ppa:nginx/stable
sudo apt update
sudo apt install nginx
sudo systemctl enable nginx
```

```
# /home/developer
git clone https://github.com/Timur00Kh/molkongress-kazan.ru-nginx.git
sudo nano /etc/nginx/nginx.conf
sudo nginx -t
sudo nginx -s reload
```

```
http {
    ...
    include /home/developer/molkongress-kazan.ru-nginx/http/*.conf;
    ...
}
```




## Nginx конфигурируем домены

```
server {
    server_name example.com;
    listen 80;

    location /.well-known {
        root /var/www/html;
    }
}

```

## Certbot устанавливаем и получаем сертификаты на домены

#### Через Snap 
В этот раз получилось с ошибкой
```
sudo apt update
sudo apt install snapd
sudo snap install core 
sudo snap refresh core

sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
# sudo certbot --nginx
sudo certbot certonly --nginx
```

#### Через apt

```
sudo apt-get install certbot

# если Nginx запущен
sudo certbot certonly --webroot

# если Nginx не запущен
sudo certbot certonly --standalone
```


## Nginx конфигурируем домены c SSL

Для статического сайта 
```
server {
    server_name molkongress-kazan.ru www.molkongress-kazan.ru;
    root /home/developer/congress-front/build;
    listen 80;
    
    location /.well-known {
        root /var/www/html;
    }
    
    location / {
        return 301 https://$host$request_uri;
    }
    
    error_log /var/log/nginx/project_front_error.log;
    access_log /var/log/nginx/project_front_access.log;
}

server {
    server_name molkongress-kazan.ru www.molkongress-kazan.ru;
    root /home/developer/congress-front/build;
    listen 443 ssl;
    
    ssl_certificate /etc/letsencrypt/live/molkongress-kazan.ru/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/molkongress-kazan.ru/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/molkongress-kazan.ru/chain.pem;

    
    location / {
        try_files $uri $uri/ =404;
    }
    
    error_log /var/log/nginx/project_front_error_ssl.log;
    access_log /var/log/nginx/project_front_access_ssl.log;
}

```

Для PHP: 
```
server {
    server_name api.molkongress-kazan.ru www.api.molkongress-kazan.ru;
    root /home/developer/congress-backend/public;
    listen 80;
    
    location /.well-known {
        root /var/www/html;
    }
    
    location / {
        return 301 https://$host$request_uri;
    }
    
    error_log /var/log/nginx/project_backend_error.log;
    access_log /var/log/nginx/project_backend_access.log;
}

server {
    server_name api.molkongress-kazan.ru www.api.molkongress-kazan.ru;
    root /home/developer/congress-backend/public;
    listen 443 ssl;

    ssl_certificate /etc/letsencrypt/live/molkongress-kazan.ru/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/molkongress-kazan.ru/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/molkongress-kazan.ru/chain.pem;

    location / {
           add_header 'Access-Control-Allow-Origin' '*';
                add_header 'Access-Control-Allow-Credentials' 'true';
                add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS, PUT, DELETE, HEAD$
                add_header 'Access-Control-Allow-Headers' 'DNT,X-Mx-ReqToken,Keep-Alive,User-Age$

           if ($request_method = 'OPTIONS') {
                add_header 'Access-Control-Max-Age' '1728000';
                add_header 'Access-Control-Allow-Credentials' 'true';
                add_header 'Access-Control-Allow-Headers' 'Origin,Content-Type,Accept,Authorizat$
                add_header 'Content-Type' 'text/plain; charset=UTF-8';
                add_header 'Content-Length' '0';
                return 204;
           }
        # try to serve file directly, fallback to index.php
        try_files $uri /index.php$is_args$args;
    }

    location ~ ^/index\.php(/|$) {
        fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
        fastcgi_split_path_info ^(.+\.php)(/.*)$;
        include fastcgi_params;

        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        fastcgi_param DOCUMENT_ROOT $realpath_root;
        internal;
    }

    # return 404 for all other php files not matching the front controller
    # this prevents access to other php files you don't want to be accessible.
    location ~ \.php$ {
        return 404;
    }

    error_log /var/log/nginx/project_backend_error_ssl.log;
    access_log /var/log/nginx/project_backend_access_ssl.log;
}
```




# Links 
+ [Nginx установка](https://losst.ru/ustanovka-nginx-ubuntu-16-04)
+ [certbot](https://certbot.eff.org/lets-encrypt/ubuntubionic-nginx)
