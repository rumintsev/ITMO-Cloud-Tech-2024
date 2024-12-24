# Лабораторная работа 1
### Задача:

Настроить nginx по заданному тз:
1. Должен работать по https c сертификатом
2. Настроить принудительное перенаправление HTTP-запросов (порт 80) на HTTPS (порт 443) для обеспечения безопасного соединения.
3. Использовать alias для создания псевдонимов путей к файлам или каталогам на сервере.
4. Настроить виртуальные хосты для обслуживания нескольких доменных имен на одном сервере.

### Ход работы:

Для начала обновим apt-get, чтобы не было ошибок при загрузке nginx `sudo apt-get update`, затем устанавливаем сам nginx `sudo apt install nginx -y`.  

Далее запускаем nginx `sudo service nginx start` и проверяем, что он работает с помощью `systemctl status nginx`.  

Переходим в `/etc/nginx/sites-available/` и создаём конфигурацию для нашего первого проекта `sudo touch helloworld.itmo`, для того чтобы открыть файл используем `sudo nano helloworld.itmo`.
```nginx
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
```nginx
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
```bash
sudo ln -s /etc/nginx/sites-available/helloworld.itmo /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/helloworld.mike /etc/nginx/sites-enabled/
```

Чтобы мы смогли открыть `helloworld.itmo` в браузере, нам нужно настроить локальные домены. Открываем `C:\Windows\System32\drivers\etc\hosts`, это путь на windows, так как я делаю всё с помощью wsl, на linux он находится в `/etc/hosts`. Далее добавляем в `hosts` строки `127.0.0.1 helloworld.itmo` и `127.0.0.1 helloworld.mike`.

Так как нам нужно чтобы всё работало по https c сертификатом, создадим самоподписанные сертификаты. Создаём директорию `sudo mkdir /etc/nginx/ssl` и создаём сертификаты (среди прочего он спросит `Common Name`, оно должно совпадать с доменом, в нашем случае `helloworld.itmo` и `helloworld.mike`):
```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/ssl/helloworld.itmo.key -out /etc/nginx/ssl/helloworld.itmo.crt
```
```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/ssl/helloworld.mike.key -out /etc/nginx/ssl/helloworld.mike.crt
```
Теперь добавим каталоги проектов
```bash
sudo mkdir /var/www/helloworld.itmo
sudo mkdir /var/www/helloworld.mike
```
В каждый добавим `index.html` с уникальным содержанием и папки `more` с уникальными доп страничками.

Далее проверим конфигурацию `sudo nginx -t` и перезапустим nginx `sudo systemctl reload nginx`.

Теперь если мы перейдём на `http://helloworld.itmo/` или `http://helloworld.mike/`мы увидим, что всё работает.

![Снимок экрана 2024-10-13 002012](https://github.com/user-attachments/assets/86a55214-94d9-41c0-8b59-ca13bfd32653)

![Снимок экрана 2024-10-13 002036](https://github.com/user-attachments/assets/21694582-f8dc-461a-a216-8d3757984d30)

Теперь проверим работоспособность `alias`.

![Снимок экрана 2024-10-13 002208](https://github.com/user-attachments/assets/078c4ed7-01ea-4336-9062-911386f6eda3)

![Снимок экрана 2024-10-13 002224](https://github.com/user-attachments/assets/8b79593f-31cb-477c-8f70-df0f567972a0)
