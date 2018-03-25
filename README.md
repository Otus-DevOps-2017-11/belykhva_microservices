# Задание 3

- Созданы образы для каждого из сервисов
- Если базовый образ использовался ранее при сборке новых образов он не скачивается повторно (образ для сервиса ui)
- Из созданных образов запущены и проверены контейнеры
- Запуск контейнеров с альтернативными сетевыми алиасами и переменным окружения:

```
docker run -d \
--network=reddit \
--network-alias=net_post_db \
--network-alias=net_comment_db \
mongo:latest
```

```
docker run -d \
--network=reddit \
--network-alias=net_post \
-e POST_DATABASE_HOST=net_post_db \
alltoday/post:1.0
```

```
docker run -d \
--network=reddit \
--network-alias=net_comment \
-e COMMENT_DATABASE_HOST=net_comment_db \
alltoday/comment:1.0
```

```
docker run -d \
--network=reddit \
-p 9292:9292 \
-e POST_SERVICE_HOST=net_post \
-e COMMENT_SERVICE_HOST=net_comment \
alltoday/ui:1.0
```

- Собран образ ui на основе Alpine Linux, Dockerfile в репозитории, результат ниже:

```
REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
alltoday/ui            3.0                 9bb76b9b20dd        17 minutes ago      209MB
alltoday/ui            2.0                 ebf2048876c9        13 hours ago        455MB
alltoday/ui            1.0                 c3975436b35c        13 hours ago        779MB
alltoday/comment       1.0                 6804fe2f4fb4        14 hours ago        771MB
alltoday/post          1.0                 84e936431ca6        14 hours ago        102MB
```

- Docker volume создан и подключен к контейнеру MongoDB

# Задание 2

- Установлен docker-machine в GCP
- Создан образ и развернут контейнер с приложением
- Образ пУшнут в Docker Hub (alltoday/otus-reddit:1.0)

- Количество слоев в контейнере можно было сократить, перечислив все команды, после RUN,  через &&.

## Задание со *

- Параметр --pid определяет namespace в котором будет работать контейнер. Определяя в качестве аргумента host мы отказываемся от изоляции процессов контейнера, процессы хоста доступны из него.

# Задание 1

- Установлен Docker
- Запущен первый контейнер
- Прочеканы списки контейнеров и образов
- Запущен контейнер в интерактивном режиме
- Запущен дополнительный процесс в работающем контейнере
- Изменения в работающем контейнере закомичены в новый образ
- Испробованы различные способы убийств контейнеров и образов

## Задание со *

- Комментарии находятся в файле docker-1.log
