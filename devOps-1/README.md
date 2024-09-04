Для начала обновим apt-get, чтобы не было ошибок при загрузке nginx `sudo apt-get update`, 
затем устанавливаем сам nginx `sudo apt install nginx -y`.  

Далее запускаем nginx `sudo service nginx start`. Проверить, что nginx работает можно с помощью `systemctl status nginx`.  

Переходим в `/etc/nginx/sites-available/` 
и создаём конфигурации для нашего сайта `sudo touch helloworld.itmo`, 
для того чтобы открыть файл используем `sudo nano helloworld.itmo`:
```
server {
    listen 80;
    server_name helloworld.itmo www.helloworld.itmo;

    root /var/www/helloworld.itmo;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }

    error_page 404 /404.html;
    location = /404.html {
        internal;
    }
}
```
Символические ссылки на файлы из `/etc/nginx/sites-available/` находятся в `/etc/nginx/sites-enabled/`, 
и Nginx использует их для определения, какие сайты должны быть активными и обслуживаться, поэтому создадим такую ссылку 
`sudo ln -s /etc/nginx/sites-available/helloWorld.itmo /etc/nginx/sites-enabled/`.  

Далее проверим конфигурацию `sudo nginx -t` и перезапустим Nginx `sudo systemctl reload nginx`.  

Если мы перейдём на `http://localhost/` мы увидим, что всё работает:
```
Welcome to nginx!
If you see this page, the nginx web server is successfully installed and working. Further configuration is required.

For online documentation and support please refer to nginx.org.
Commercial support is available at nginx.com.

Thank you for using nginx.
```
Теперь нам нужно чтобы всё работало по https c сертификатом. 
Для этого создадим самоподписанный сертификат: создаём директорию `sudo mkdir /etc/nginx/ssl` и создаём сертификат 
(среди прочего он спросит `Common Name`, оно должно совпадать с доменом, в нашем случае `helloworld.itmo`):
```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/ssl/helloworld.itmo.key -out /etc/nginx/ssl/helloworld.itmo.crt
```
Изменим конфигурацию:
```
server {
    listen 80;
    server_name helloworld.itmo www.helloworld.itmo;

    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name helloworld.itmo www.helloworld.itmo;

    ssl_certificate /etc/nginx/ssl/helloworld.itmo.crt;
    ssl_certificate_key /etc/nginx/ssl/helloworld.itmo.key;

    root /var/www/helloworld.itmo;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    error_page 404 /404.html;
    location = /404.html {
        internal;
    }
}
```
Проверяем, всё ли окей `sudo nginx -t` и перезапускаем `sudo systemctl reload nginx`.  

Если мы перейдём на `https://localhost/` мы увидим, что всё работает, хоть и выдаёт 404:
```
404 Not Found
nginx/1.18.0 (Ubuntu)
```
