# Monitoring

## Лабораторная работа №5

# Цель работы

Сделать мониторинг сервиса поднятого в k8s с помощью Prometheus и Grafana. Показать 2 рабочих графика, прикрепить скрины процесса.

# Теоретическая часть

Архитектура Prometheus:

1. Prometheus Server: 

-Сбор метрик через HTTP-запросы 
- Хранение временных рядов в локальной БД (TSDB)
- Обработка запросов через PromQL

2. Экспортеры:

- Преобразуют данные из различных систем в формат Prometheus
- Примеры: node_exporter (метрики сервера), blackbox_exporter (мониторинг HTTP)

3. Pushgateway:

- Промежуточное звено для кратковременных задач

4. Alertmanager:

- Обработка и маршрутизация алертов

Как работает:
Приложение предоставляет /metrics
раз в scrape_interval (где-то 15–60 сек) Prometheus отправляет HTTP-запрос к /metrics
prometheus сохраняет метрики в TSDB (Time-Series Database) - данные хранятся в виде временных рядов. Каждая запись имеет - метки, значение и временную метку, когда была собрана запись. 

Grafana:

Запрос данных из источников через их API (напрмиер prometheus) -->  Преобразование сырых данных в визуализации --> Настройка  панелей с фильтрацией

# Ход работы

Возьмем поднятый k8s кластер из прошлой лабораторной и попробуем подключить для него Prometheus и Grafana. 

Для начала установим helm:

![image](https://github.com/user-attachments/assets/e814b4c8-3512-4e45-8e05-3ea9ca55ad3c)

После создадим namespace monitoring - создается для изоляции и организации ресурсов, связанных с системой мониторинга. 

`kubectl create ns monitoring`

После устанавливаем kube-prometheus-stack:

![image](https://github.com/user-attachments/assets/5e2a1d6b-1066-4a5a-8e19-689de671236f)

Проверяем поднялись ли поды - `kubectl get pods n monitoring -w`:

![image](https://github.com/user-attachments/assets/e843ca68-9478-451e-a1e3-f8b3e3d6149b)

Проверяем серсисы - `kubectl get svc -n monitoring`:

![image](https://github.com/user-attachments/assets/17958ad1-b897-441a-85f5-dbcbe00e5f99)

Теперь нужно добавить сбор метрик:

![image](https://github.com/user-attachments/assets/7cf2a844-1764-4ab1-b8f4-2ab2d679fdf5)

Добавить аннотации в deployment.yaml:

![image](https://github.com/user-attachments/assets/e7eb56d9-bb07-45d7-a6cf-d9d82057de65)

И также создать файл fastapi-servicemonitor.yaml - создается для настройки автоматического обнаружения и сбора метрик Prometheus с FastAPI-приложения в k8s:

![image](https://github.com/user-attachments/assets/7a65e8f4-2fe1-4239-84ff-a2ed01429e14)

Как это работает: Prometheus Operator обнаруживает ServiceMonitor, сопоставляет селекторы с сервисами Kubernetes:

- Ищет сервисы с лейблом app: fastapi
- Проверяет, что у сервиса есть порт с именем http

Добавляет цели в список scrape-конфигураций Prometheus

После всего пересобираем образ, применяем политику:

`kubectl apply -f fastapi-servicemonitor.yaml`

После через NodePort заходим в браузере в prometheus и grafana:

![image](https://github.com/user-attachments/assets/b51aaba7-fa85-445d-8f93-eede288c4ffb)

В таргетах видно, что кластер поднят и есть наш мониторинг.

![image](https://github.com/user-attachments/assets/71c75453-d0a3-4558-a04a-e2908c7fd6e6)

После в Grafana добавляем новый источник данных, ссылаясь на NodeIp с тем же портом 30090.

![image](https://github.com/user-attachments/assets/72a71d94-95ce-4a98-b622-2a1311ca1de7)

После добавляем новый дашборд, для примера импортируем готовый ID 11074 - FastAPI Monitoring, что включает HTTP-запросыь, время ответа, использование CPU/памяти. А также ID 6417 - Kubernetes Pod Details. 

![image](https://github.com/user-attachments/assets/15ba669b-a773-481b-8d5a-66571fa5f67d)

![image](https://github.com/user-attachments/assets/9a50e510-9e0f-4772-98df-51fff4d82267)

Также можно создавать собственные дашборды:

![image](https://github.com/user-attachments/assets/1ff8aa38-0b87-4c2f-b25a-1cd8cc2b0c7b)



