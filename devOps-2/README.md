Плохой
```
# Используем тяжелый образ Ubuntu и latest-тег
FROM ubuntu:latest  

# Устанавливаем пакеты, не фиксируя версии и не удаляя лишний кэш
RUN apt-get update && apt-get install -y \
    curl \
    git \

# Жёстко встраиваем секреты в образ
ENV API_KEY="super_secret_key"
```

Хороший
```
# Используем легковесный базовый образ Alpine
FROM python:3.9-alpine

# Добавляем зависимости в отдельном слое и фиксируем версии
RUN apk add --no-cache --update \
    curl=7.78.0-r0 \
    git=2.32.0-r0

Используем внешние переменные окружения для секретов
# Например: docker run -e API_KEY="super_secret_key" myapp
```
Альтернатива к первой плохой практике - использовать стабильную версию Ubuntu вместо latest (`FROM ubuntu:20.04`), так мы избегаем неожиданного изменения базового образа

Альтернатива к второй плохой практике - при обновление пакетов использовать apt-get clean для уменьшения размера образа и удалять `lists`
```
RUN apt-get update && \
    apt-get install -y \
        curl \
        git \
        python3 \
        python3-pip && \
    apt-get clean && rm -rf /var/lib/apt/lists/*
```
из статьи:
Приложение не должно иметь встроенной конфигурации. Cейчас есть куча решений для этого и большинство систем кластеризации и развертывания могут работать с решениями для загрузки конфигурации при старте (configmaps, zookeeper, consul, etc) и секреты (vault,  keywhiz, confidant, cerberus).
