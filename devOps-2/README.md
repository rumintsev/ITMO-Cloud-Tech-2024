## 3 плохие практики по написанию dockerFile
Сразу напишем плохой dockerfile, а в комментариях к коду, в чём заключается эта плохая практика
```dockerfile
# 1. Используем тяжелый образ Ubuntu и latest-тег
FROM ubuntu:latest  

# 2. Устанавливаем пакеты, не фиксируя версии и не удаляя лишний кэш
RUN apt-get update && apt-get install -y \
    curl \
    git

# 3. Жёстко встраиваем секреты в образ
ENV API_KEY="super_secret_key"
```

Далее напишем хороший dockerfile, а в комментариях к коду, как была решена плохая практика
```dockerfile
# 1. Используем легковесный базовый образ Alpine с указанной версией
FROM python:3.9-alpine

# 2. Устанавливаем пакеты с no-cache и явно указываем версии
RUN apk add --no-cache --update \
    curl=8.11.0-r2 \
    git=2.45.2-r0
```
```bash
# 3. Используем внешние переменные окружения для секретов, например при запуске
docker run -e API_KEY="super_secret_key" myapp
```
**Комментарий к 1 практике:**

Большие образы, такие как ubuntu, увеличивают время загрузки и использование ресурсов (Alpine ~5 МБ, Ubuntu 29–40 МБ), поэтому для неспецифических задач используем Alpine. А указание версии позволяет всё делать на стабильной версии.

**Комментарий к 2 практике:**

Если мы не указываем версию явно это может привести к траблам при сборке или запуске, но нужно быть аккуратным, так как в старых версиях потенциально могут быть уязвимости. --no-cache же позволяет не захламлять хранилище лишним кэшэм.

**Комментарий к 3 практике:**

Если добавлять секрет прямо в Dockerfile мы рискуем, что он утечёт при разных ситуациях, чтобы такого точно не случилось и чтобы лишний раз об этом не думать лучше передавать их контейнеру на этапе запуска, а не сборки.

**Альтернативное решение к 1 плохой практике:**

Использовать стабильную версию Ubuntu вместо latest (`FROM ubuntu:20.04`), так мы будем собирать контейнеры на стабильной версии

**Альтернативное решение к 2 плохой практике:**

Если вам нужны библиотеки, которые отсутствуют в Alpine можно использовать apt-get clean и удалять `lists`, плюс не забываем про указание версии
```dockerfile
RUN apt-get update && \
    apt-get install -y \
        curl=7.68.0-1ubuntu2.12 \
        git=1:2.25.1-1ubuntu3.10 && \
    apt-get clean && rm -rf /var/lib/apt/lists/*
```
## 2 плохие практики по работе с контейнерами

**1. Запуск контейнера без ограничений по CPU и памяти:**

Если контейнер написан неправильно или случилось что-то непредвиденное, один контейнер может занять всю оперативную память или процессорное время, что приведёт к неприятному исходу.

Для предотвращения таких неприятных вещей, при запуске ограничиваем контейнер в ресурсах
```bash
docker run --memory=512m --cpus="1.5" your_image
```

**2. Запуск контейнеров с правами root:**

При запуске от имени root может произойти выход за пределы контейнера или могут быть выполнены неприятные операции с файловой системой, сетью и всяким таким.

Для предотвращения таких неприятных вещей, при запуске выбираем юзера, не имеющего административных прав
```bash
docker run --user 1000:1000 -it your_image
```
## Соберём образы и посмотрим

Создадим два докерфайла вставим в них наши хорошие и плохие практики

![image](https://github.com/user-attachments/assets/024d9943-8f69-4b54-92c1-772856416c8a)

Для определения актуальных версий для хорошего докерфайла можно воспользоваться 
```bash
docker run -it python:3.9-alpine sh`
```
```sh
apk update
apk search curl
apk search git

```
![image](https://github.com/user-attachments/assets/c4fa61f1-f8a9-438a-b85a-075d53f79bc4)

Билдим оба образа

![image](https://github.com/user-attachments/assets/eb52169a-cb19-43c9-8aaf-5020e0d53585)
![image](https://github.com/user-attachments/assets/9d066ca9-d288-4db6-9b95-c15d0756f239)

Помимо повышения безопасности хорошего докерфайла отметим и существенное уменьшение размера хорошего образа, чего мы и хотели добиться

![image](https://github.com/user-attachments/assets/3a812a5b-68f5-491b-b066-164e0db74d9e)
