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
