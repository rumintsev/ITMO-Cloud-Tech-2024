## 5 плохих практик по написанию CI/CD
Есть такой GitHub Actions и для него пишут ямлики (yaml файлики) для ci/cd, давайте напишем такой плохой и хороший
(файлики не очень реальные, но зато в них есть все нам нужные практики)

Плохой:
```yaml
name: BadBoy
on:
  push:
    branches:
      - main
jobs:

  deploy:
    runs-on: ubuntu-latest
    steps:
      # 1. Хардкодинг секретов
      - name: Deploy
        run: |
          ssh -i "secret-key" user@gmail.com 'git pull origin main && npm install && pm2 restart all'

  build:
    runs-on: ubuntu-latest
    steps:
      # 2. Отсутствие кеширования
      - name: Install without cache
        run: npm install

      # 3. Отсутствие таймаутов
      - name: No Timeout
        run: |
          echo "Task started, will be ready in infinity :)"

      # 4. Нефиксированные версии зависимостей
      - name: Use without version
        uses: actions/setup-node@latest

  all-at-once:
    runs-on: ubuntu-latest
    steps:
      # 5. Перегруженный пайплайн или отсутствие имён
      - name: Checkout Code
        uses: actions/checkout@v2
      - name: A lot of tasks
        run: npm install
        run: npm run test
      - name: Lint Code
        run: npm run lint
      - name: Build Application
        run: npm run build
```
А теперь давайте исправим плохие практики в нём и напишем хороший
```yaml
name: GoodBoy
on:
  push:
    branches:
      - main
jobs:

  deploy:
    runs-on: ubuntu-latest
    steps:
      # 1. Безопасное использование секретов
      - name: Setup SSH
        uses: webfactory/ssh-agent@v0.5.3
        with:
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
      - name: Deploy
        run: |
          ssh user@gmail.com 'git pull origin main && npm install && pm2 restart all'

  build:
    runs-on: ubuntu-latest
    steps:
      # 2. Кеширование зависимостей
      - name: Use cache
        uses: actions/cache@v2
        with:
          path: node_modules
          key: ${{ runner.os }}-node-${{ hashFiles('package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node
      - name: Install 
        run: npm install

      # 3. Установка таймаутов
      - name: Task with Timeout
        run: |
          echo "Task started, be in 10 min :)"
        timeout-minutes: 10

      # 4. Фиксирование версий зависимостей
      - name: Use with version
        uses: actions/setup-node@v3
        with:
          node-version: '16'

      # 5. Разбиение на шаги и спользование имён
  checkout:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout 
        uses: actions/checkout@v2

  install:
    runs-on: ubuntu-latest
    needs: checkout
    steps:
      - name: Install 
        run: npm install

  test-and-lint:
    runs-on: ubuntu-latest
    needs: install
    steps:
      - name: Tests 
        run: npm run test
      - name: Lint 
        run: npm run lint

  build:
    runs-on: ubuntu-latest
    needs: test-and-lint
    steps:
      - name: Build 
        run: npm run build
```
**Комментарий к 1 практике:**

Используя GitHub Secrets вместо хардкодинга секретов мы очевидно не рискуем, что что-то важное, что не должно утечь, утечёт.

**Комментарий к 2 практике:**

Кешируя зависимости мы ускореем процесс установки и предотвращаем повторное скачивание пакетов.

**Комментарий к 3 практике:**

Если мы запустим задачу, из-за ошибки в которой, она будет длится бесконечно долго, все будут грустить, поэтому устанавливаем таймаут.

**Комментарий к 4 практике:**

Вот что-то обновилось, и всё у нас сломалось, поэтому на всякий случай фиксируем версии для более надёжной и предсказуемой работы

**Комментарий к 5 практике:**

Используем имена и разбиение во первых для читаемости, но не этим единым, 
если бы мы разделили `test` и `lint` на две джобы, то они по идеи выполнялись бы параллельно, так что это делается 
как для читаемости, так и для управляемости (с помощью `needs`) и оптимизации.
