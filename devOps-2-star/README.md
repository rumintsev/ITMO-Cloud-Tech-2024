# Лабораторная работа 2 star
### Задачи:

1. Написать “плохой” Docker compose файл, в котором есть не менее трех “bad practices” по их написанию
2. Написать “хороший” Docker compose файл, в котором эти плохие практики исправлены
3. В Readme описать каждую из плохих практик в плохом файле, почему она плохая и как в хорошем она была исправлена, как исправление повлияло на результат
4. После предыдущих пунктов в хорошем файле настроить сервисы так, чтобы контейнеры в рамках этого compose-проекта так же поднимались вместе, но не "видели" друг друга по сети. В отчете описать, как этого добились и кратко объяснить принцип такой изоляции

### Ход работы:

Для начала проверим, установлен ли docker compose, для чего вводим
```
docker compose version
```

Теперь составим docker compose с плохими практиками:

```yaml
services:
  app_1:
    # 1. Тег latest
    image: ubuntu:latest
    command: ["sleep", "infinity"]
    # 2. Привилегии
    privileged: true
    # 3. Секреты
    environment:
      - SECRET_KEY=secret_key_lol
    # 4. Отсутствие ограничений по ресурсам

  app_2:
    image: ubuntu:latest
    command: ["sleep", "infinity"]
```
Запускаем плохую версию:

![Снимок экрана 2024-12-22 023659](https://github.com/user-attachments/assets/832d935b-d98d-46b9-979e-17618aba852b)

Теперь составим docker compose с хорошими практиками:

```yaml
services:
  app_1:
    # 1. Указываем версию ubuntu
    image: ubuntu:22.04
    command: ["sleep", "infinity"]
    # 2. Не используем привилегии, когда это не нужно
    privileged: false
    # 3. Ограничиваем ресурсы
    deploy:
      resources:
        limits:
          memory: 256M
          cpus: "0.5"
    secrets:
      - app_1_secret
    # 4. Не пишем секреты в коде
    environment:
      SECRET_KEY_FILE: /run/secrets/app_1_secret

  app_2:
    image: ubuntu:22.04
    command: ["sleep", "infinity"]
    privileged: false
    deploy:
      resources:
        limits:
          memory: 256M
          cpus: "0.5"

secrets:
  app_1_secret:
    file: ./secrets/app_1_secret.txt
```
Запускаем хорошую версию:

![Снимок экрана 2024-12-22 023547](https://github.com/user-attachments/assets/38969a9a-5ac6-452f-a276-4c7cc8dfe945)

С помощью `docker ps` можем проверить, что два контейнера действительно работают:

![Снимок экрана 2024-12-22 022843](https://github.com/user-attachments/assets/32db3b26-bd05-43a4-962d-fa09a2b12052)

### Комментарий к 1 практике. Версия Ubuntu.

Использование тега latest создаёт неопределенность, так как может меняться с каждым обновлением.
Была указана версия, что позволяет гарантировать стабильность работы контейнера, независимо от обновлений.

### Комментарий к 2 практике. Привилегированный доступ.

Привилегированный доступ даёт почти полный доступ к хостовой системе, что создаёт большие риски безопасности.
Ограничили доступ контейнера к хостовой системе, придерживаясь принципа минимальных привилегий.

### Комментарий к 3 практике. Ограничений на ресурсы.

Без ограничения ресурсов контейнеры могут потреблять слишком много ресурсов, что может вызвать сбои в работе всей системы.
Были добавлены ограничения, что предотвращает избыточное потребление.

### Комментарий к 4 практике. Секреты в переменных окружения.

Хранение секретов в переменных окружения может привести к утечке этих данных.
Секреты были вынесены в отдельный файл, что может защитить чувствительные данные от утечки.

## Реализация изоляции

Чтобы изолировать контейнеры, создадим две отдельные сети и подключим сервисы к ним.
```yaml
services:
  app_1:
    image: ubuntu:22.04
    command: ["sleep", "infinity"]
    privileged: false
    deploy:
      resources:
        limits:
          memory: 256M
          cpus: "0.5"
    secrets:
      - app_1_secret
    environment:
      SECRET_KEY_FILE: /run/secrets/app_1_secret
    networks:
      - network1

  app_2:
    image: ubuntu:22.04
    command: ["sleep", "infinity"]
    privileged: false
    deploy:
      resources:
        limits:
          memory: 256M
          cpus: "0.5"
    networks:
      - network2

secrets:
  app_1_secret:
    file: ./secrets/app_1_secret.txt

networks:
  network1:
    driver: bridge
  network2:
    driver: bridge
```

Подключаемся к каждому контейнеру и пингуем другой:

![Снимок экрана 2024-12-22 030933](https://github.com/user-attachments/assets/b1847fe3-7102-4a92-83dc-e0b485c6a086)

![Снимок экрана 2024-12-22 031552](https://github.com/user-attachments/assets/9355e582-ad5a-40f5-998a-7e0385a671fd)

Отлично, изоляция реализована успешно :)
