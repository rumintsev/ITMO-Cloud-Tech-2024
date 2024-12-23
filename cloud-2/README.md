# Лабораторная работа 2 (4 вариант)

### Сравнение сервисов Amazon Web Services и Microsoft Azure. Создание единой кросс-провайдерной сервисной модели.

**Цель работы:** Получение навыков аналитики и понимания спектра публичных облачных сервисов без привязки к вендору. Формирование у студентов комплексного видения Облака. 

**Дано:** Слепок данных биллинга от провайдера после небольшой обработки в виде SQL-параметров. Символ % в начале/конце означает, что перед/после него может стоять любой набор символов.
Образец итогового соответствия, что желательно получить в конце. В этом же документе  

**Необходимо:** Импортировать файл .csv в Excel или любую другую программу работы с таблицами. Для Excel делается на вкладке Данные – Из текстового / csv файла – выбрать файл, разделитель – точка с запятой.
Распределить потребление сервисов по иерархии, чтобы можно было провести анализ от большего к меньшему (напр. От всех вычислительных ресурсов Compute дойти до конкретного типа использования - Выделенной стойка в датацентре Dedicated host usage). При этом сохранять логическую концепцию, выработанную в Лабораторной работе 1.
Сохранить файл и залить в соответствующую папку на Google Drive.

**Алгоритм работы:** Сопоставить входящие данные от провайдера с его же документацией. Написать в соответствие колонкам справа значения 5 колонок слева, которые бы однозначно классифицировали тип сервиса. Для столбцов IT Tower и Service Family значения можно выбрать из образца. В ходе выполнения работы не отходить от принципов классификации, выбранных в Лабораторной работе 1. Например, если сервис Машинного обучения был разбит на Вычислительные мощности и Облачные сервисы, то продолжать его разбивать и в новых данных.

### Ход работы

1. Импортируем данные из `.csv` в excel и сортируем для более удобного взаимодействия.

![Снимок экрана 2024-12-21 012537](https://github.com/user-attachments/assets/810cd95c-64e5-4047-aeaf-08dee95ee654)

2. Знакомимся как с официальными ресурсами, где можно узнать подробнее про сервисы, так и с неофициальными

Серависы:

### Azure Data Factory
Платформа для интеграции данных. Позволяет перемещать данные , обрабатывать их и автоматизировать процессы.

### Azure Database for PostgreSQL
Управляемая база данных на основе PostgreSQL. Предоставляет масштабируемость, безопасность, высокую доступность и автоматические бэкапы.

### Azure Databricks
Инструмент для обработки больших данных и машинного обучения. Предназначен для аналитиков, разработчиков и инженеров данных.

### Azure Firewall
Управляемый брандмауэр для защиты виртуальных сетей. Предоставляет функции фильтрации входящего и исходящего трафика.

### Bandwidth
Используется для учета объема данных, переданных через интернет или виртуальные сети.

### Cache (Azure Redis Cache)
Высокопроизводительное кеширование. Предназначено для улучшения производительности приложений.

### CDN (Content Delivery Network)
Сеть доставки контента для ускорения загрузки веб-ресурсов (изображений, видео, CSS, JavaScript и т. д.) через распределенные серверы, которые физически ближе к пользователям.

### Container Instances
Сервис для развертывания и управления контейнерами без необходимости управлять виртуальными машинами. Подходит для тестирования, запуска краткосрочных задач или работы с микросервисами.

### Content Delivery Network (CDN)
Улучшает скорость доставки контента, благодаря географически распределенным серверам.

### Machine Learning Studio
Платформа для разработки, тестирования и развертывания моделей машинного обучения.

### Notification Hubs
Сервис для отправки push-уведомлений на устройства с iOS, Android, Windows.

### Power BI
Инструмент для бизнес-аналитики, позволяющий визуализировать данные, создавать отчеты и дашборды, а также анализировать их в реальном времени.

### Power BI Embedded
Версия Power BI, предназначенная для встраивания аналитических возможностей в приложения или сайты.

### Redis Cache
Обеспечивает быстрый доступ к данным и улучшает отклик приложений.

### Traffic Manager
Сервис DNS для балансировки трафика между несколькими конечными точками.

### VPN Gateway
Сервис для создания защищенного соединения.

3. На основе найденных данных заполняем таблицу, которую также можно посмотреть по [ссылке](https://docs.google.com/spreadsheets/d/1f_naWiF6IUNCaVyvCeK4lQqdV1EZQ9iGGqnlMGxNQfc/edit?usp=sharing)

![image](https://github.com/user-attachments/assets/fa073af8-c51e-42ee-90d6-54f588e3fe64)

### Итог

Познакомившись с Azure, мы приобрели навыки аналитики и изучили спектр облачных сервисов, что способствовало формированию комплексного понимания концепции.