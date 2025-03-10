# Лабораторная работа 3
### Задачи:

1. Написать “плохой” CI/CD файл, который работает, но в нем есть не менее пяти “bad practices” по написанию CI/CD
2. Написать “хороший” CI/CD, в котором эти плохие практики исправлены
3. В Readme описать каждую из плохих практик в плохом файле, почему она плохая и как в хорошем она была исправлена, как исправление повлияло на результат
4. Прочитать историю про Васю (она быстрая, забавная и того стоит): https://habr.com/ru/articles/689234/

### Ход работы:

## 5 плохих практик по написанию CI/CD
Зайдём в GitHub Actions и напишем в нём два ямлика (yaml файлика) для ci/cd (что будет происходить, например, при пуше в репозиторий) - плохой и хороший

Плохой:
```yaml
name: BadBoy
on:
  push:
    branches:
      - main
jobs:

  main:
      # 1. Использование тэга latest
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
      # 2. Использование неактуальной версии checkout
        uses: actions/checkout@v1
        
      # 3. Отсутствие кэширования
      - name: Install dependencies
        run: npm ci
        
      # 4. Перегруженный пайплайн
      - name: Run all at once
      # 5. Отсутствие таймаутов
        run: |
          npm run test || echo "Tests failed"
          npm run lint || echo "Linting failed"
          npm run build
```
Проверим работоспособность

![image](https://github.com/user-attachments/assets/4126bbeb-f17b-44c6-9754-0ca9cf70dd98)

Видим, что даже GitHub ругается 

![image](https://github.com/user-attachments/assets/fd584147-8bf8-4915-a114-a19e8144a903)

А теперь давайте исправим плохие практики и напишем хороший
```yaml
name: GoodBoy
on:
  push:
    branches:
      - main
jobs:

  main:
      # 1. Используем определённую версию
    runs-on: ubuntu-24.04
    steps:
      - name: Checkout
      # 2. Использование актуальной версии checkout
        uses: actions/checkout@v3

      # 3. Кеширование зависимостей
      - name: Use cache
        uses: actions/cache@v4
        with:
          path: node_modules
          key: ${{ runner.os }}-node-${{ hashFiles('package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node

      - name: Install dependencies
        run: npm ci

      # 4. Разбиение на шаги
      - name: Run tests
        run: npm run test || echo "Tests failed"
      # 5. Установка таймаутов
        timeout-minutes: 10

      - name: Run linting
        run: npm run lint || echo "Linting failed"

      - name: Build the project
        run: npm run build
```
Запускаем и всё хорошо выполняется

![image](https://github.com/user-attachments/assets/f18a6302-f74a-421f-86fb-28f5486b1533)

**Комментарий к 1 практике:**

Версия Ubuntu с тегом latest может быть обновлена в любой момент, что может повлиять на стабильность и предсказуемость. 
Указание версии гарантирует, что сборка будет выполняться в одинаковом окружении каждый раз.

**Комментарий к 2 практике:**

Новые версии экшенов включают исправления багов, улучшения безопасности и новые возможности. 
Например, версия v3 может поддерживать новые функции GitHub Actions, такие как улучшенные возможности кеширования, что делает пайплайн более эффективным.

**Комментарий к 3 практике:**

Кешируя зависимости мы ускореем процесс установки и предотвращаем повторное скачивание пакетов.

**Комментарий к 4 практике:**

Разбиваем, чтобы добиться читаемости, удобства отладки и гибкости. Разбиение позволяет легче изменять или добавлять новые этапы без нарушения общей структуры.

**Комментарий к 5 практике:**

Если мы запустим задачу, из-за ошибки в которой, она будет длится бесконечно долго, все будут грустить, поэтому устанавливаем таймаут.
