Для начала обновим apt-get, чтобы не было ошибок при загрузке nginx `sudo apt-get update`, затем устанавливаем сам nginx `sudo apt install nginx -y`.  

Далее запускаем nginx `sudo service nginx start` и проверяем, что он работает с помощью `systemctl status nginx`.  

Переходим в `/etc/nginx/sites-available/` и создаём конфигурацию для нашего первого проекта `sudo touch helloworld.itmo`, для того чтобы открыть файл используем `sudo nano helloworld.itmo`.
```
server {
    listen 80;
    server_name helloworld.itmo;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name helloworld.itmo;

    ssl_certificate /etc/nginx/ssl/helloworld.itmo.crt;
    ssl_certificate_key /etc/nginx/ssl/helloworld.itmo.key;

    root /var/www/helloworld.itmo;
    index index.html;

    location /more {
        alias /var/www/helloworld.itmo/more/;
    }
}
```
Затем создаём конфигурацию для нашего второго проекта `sudo touch helloworld.mike`.
```
server {
    listen 80;
    server_name helloworld.mike;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name helloworld.mike;

    ssl_certificate /etc/nginx/ssl/helloworld.mike.crt;
    ssl_certificate_key /etc/nginx/ssl/helloworld.mike.key;

    root /var/www/helloworld.mike;
    index index.html;

    location /more {
        alias /var/www/helloworld.mike/more/;
    }
}
```
Символические ссылки на файлы из `/etc/nginx/sites-available/` находятся в `/etc/nginx/sites-enabled/`, и nginx использует их для определения, какие сайты должны быть активными и обслуживаться, поэтому создадим такие ссылки.
```
sudo ln -s /etc/nginx/sites-available/helloworld.itmo /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/helloworld.mike /etc/nginx/sites-enabled/
```

Чтобы мы смогли открыть `helloworld.itmo` в браузере, нам нужно настроить локальные домены. Открываем `C:\Windows\System32\drivers\etc\hosts`, это путь на windows, так как я делаю всё с помощью wsl, на linux он находится в `/etc/hosts`. Далее добавляем в `hosts` строки `127.0.0.1 helloworld.itmo` и `127.0.0.1 helloworld.mike`.

Так как нам нужно чтобы всё работало по https c сертификатом, создадим самоподписанные сертификаты. Создаём директорию `sudo mkdir /etc/nginx/ssl` и создаём сертификаты (среди прочего он спросит `Common Name`, оно должно совпадать с доменом, в нашем случае `helloworld.itmo` и `helloworld.mike`):
```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/ssl/helloworld.itmo.key -out /etc/nginx/ssl/helloworld.itmo.crt
```
```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/ssl/helloworld.mike.key -out /etc/nginx/ssl/helloworld.mike.crt
```
Теперь добавим каталоги проектов
```
sudo mkdir /var/www/helloworld.itmo
sudo mkdir /var/www/helloworld.mike
```
В каждый добавим `index.html` с уникальным содержанием и папки `more` с уникальными доп страничками.

Далее проверим конфигурацию `sudo nginx -t` и перезапустим nginx `sudo systemctl reload nginx`.

Теперь если мы перейдём на `http://helloworld.itmo/` или `http://helloworld.mike/`мы увидим, что всё работает.

![image](https://github.com/user-attachments/assets/5137f042-e372-45ad-a808-1aeaa8b00aed)

![image](https://github.com/user-attachments/assets/452509e7-edee-45fc-ba27-7ef6bac06a13)

Теперь проверим работоспособность `alias`.

![image](https://github.com/user-attachments/assets/9c798738-c502-4f07-85b1-df8d8c7bc259)

![image](https://github.com/user-attachments/assets/527d7ab1-ec9c-47a4-b153-4c53094292c0)
