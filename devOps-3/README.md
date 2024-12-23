# Лабораторная работа 3
### Задачи:

1. Написать “плохой” CI/CD файл, который работает, но в нем есть не менее пяти “bad practices” по написанию CI/CD
2. Написать “хороший” CI/CD, в котором эти плохие практики исправлены
3. В Readme описать каждую из плохих практик в плохом файле, почему она плохая и как в хорошем она была исправлена, как исправление повлияло на результат
4. Прочитать историю про Васю (она быстрая, забавная и того стоит): https://habr.com/ru/articles/689234/

### Ход работы:

## 5 плохих практик по написанию CI/CD
Зайдём в GitHub Actions и напишем в нём два ямлика (yaml файлика) для ci/cd (что будет происходить, например, при пуше в репозиторий) - плохой и хороший
(файлики не очень реальные, но зато в них есть все нам нужные практики)

Плохой:
```yaml
name: BadBoy
on:
  push:
    branches:
      - main
jobs:

  build:
      # 1. Использование тэга latest
    runs-on: ubuntu-latest
    steps:
      # 2. Отсутствие кеширования
      - name: Install without cache
        run: |
          npm init -y
          npm install

      # 3. Отсутствие таймаутов
      - name: No Timeout
        run: |
          echo "Task started, will be ready in infinity :)"
          sleep 5

      # 4. Использование неактуальной версии
      - name: Checkout 
        uses: actions/checkout@v1 

  all-at-once:
    runs-on: ubuntu-latest
    steps:
      # 5. Перегруженный пайплайн
      - name: Overloaded Pipeline
        run: |
          npm init -y
          npm install
          npm run test || echo "Tests failed"
          npm run lint || echo "Linting failed"
          npm run build || echo "Build failed"
```
При запуске такого даже GitHub говорит, что не всё хорошо

![Снимок экрана 2024-12-21 211820](https://github.com/user-attachments/assets/5217184c-cc47-43f9-8ec8-98ad164847c7)

А теперь давайте исправим плохие практики в нём и напишем хороший
```yaml
name: GoodBoy
on:
  push:
    branches:
      - main
jobs:

  build:
      # 1. Используем определённую версию
    runs-on: ubuntu-24.04
    steps:
      # 2. Кеширование зависимостей
      - name: Use cache
        uses: actions/cache@v4
        with:
          path: node_modules
          key: ${{ runner.os }}-node-${{ hashFiles('package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node
      - name: Initialize npm
        run: npm init -y
      - name: Install dependencies
        run: npm install

      # 3. Установка таймаутов
      - name: Task with Timeout
        run: |
          echo "Task started, be in 10 min :)"
        timeout-minutes: 10

      # 4. Использование актуальной версии
      - name: Checkout 
        uses: actions/checkout@v3

      # 5. Разбиение на шаги
  checkout:
    runs-on: ubuntu-24.04
    steps:
    - name: Initialize npm
      run: npm init -y

    # Не забываем кэшировать
    - name: Use cache
      uses: actions/cache@v4
      with:
        path: node_modules
        key: ${{ runner.os }}-node-${{ hashFiles('package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-node

    - name: Install dependencies
      run: npm install

    - name: Run tests
      run: npm run test || echo "Tests failed"

    - name: Run linting
      run: npm run lint || echo "Linting failed"

    - name: Build the project
      run: npm run build || echo "Build failed"
```
Запускаем и всё хорошо выполняется

![Снимок экрана 2024-12-21 211709](https://github.com/user-attachments/assets/21185519-ea44-4c50-aa45-f506d7130161)

**Комментарий к 1 практике:**

Версия Ubuntu с тегом latest может быть обновлена в любой момент, что может повлиять на стабильность и предсказуемость. 
Указание версии гарантирует, что сборка будет выполняться в одинаковом окружении каждый раз.

**Комментарий к 2 практике:**

Кешируя зависимости мы ускореем процесс установки и предотвращаем повторное скачивание пакетов.

**Комментарий к 3 практике:**

Если мы запустим задачу, из-за ошибки в которой, она будет длится бесконечно долго, все будут грустить, поэтому устанавливаем таймаут.

**Комментарий к 4 практике:**

Новые версии экшенов включают исправления багов, улучшения безопасности и новые возможности. 
Например, версия v3 может поддерживать новые функции GitHub Actions, такие как улучшенные возможности кеширования, что делает пайплайн более эффективным.

**Комментарий к 5 практике:**
Разбиваем, чтобы добиться читаемости, удобства отладки и гибкости. Разбиение позволяет легче изменять или добавлять новые этапы без нарушения общей структуры.
