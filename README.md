# Задание 6

emptyyy

# Задание 4

- Вывод ifconfig для хоста и контейнера запущенного в сетевом пространстве хоста одинаковы
- Запуск одного и того же контейнера несколько раз в едином сетевом пространстве хоста приведет только к 1 одному работающему контейнеру. Сервисы в остальных не смогут подняться из-за конфликта портов.
- Namespaces на Docker-host машине:

```
$ docker-machine ssh docker-host sudo ip netns
RTNETLINK answers: Invalid argument
RTNETLINK answers: Invalid argument
35edb16bf202 (id: 1)
a34cb396a205 (id: 2)
9f34baa67ad0 (id: 3)
090f2392b6cf (id: 0)
netns
default
```
- Вывод ip a для одного из namespace:

```
$ docker-machine ssh docker-host sudo ip netns exec 35edb16bf202 ip a
RTNETLINK answers: Invalid argument
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
203: eth0@if204: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:13:00:04 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.19.0.4/16 brd 172.19.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

- Bridge Network Driver

Cписок сетей на хосте:

```
$ sudo docker network ls
NETWORK ID          NAME                         DRIVER              SCOPE
541b26de9489        back_net                     bridge              local
8e988faf3209        bridge                       bridge              local
466e46bb3d01        front_net                    bridge              local
3d65f2f2e154        host                         host                local
58458fb1461d        none                         null                local
ebdcdbcad8cc        reddit                       bridge              local
81d1b05d2f57        redditmicroservices_reddit   bridge              local
```

Список bridge на хосте:

```
$ ifconfig | grep br
br-466e46bb3d01 Link encap:Ethernet  HWaddr 02:42:a0:04:a1:eb  
br-541b26de9489 Link encap:Ethernet  HWaddr 02:42:a9:ed:4d:bf  
br-81d1b05d2f57 Link encap:Ethernet  HWaddr 02:42:b2:e1:8b:ef  
br-ebdcdbcad8cc Link encap:Ethernet  HWaddr 02:42:30:8e:49:2e
```

Сети front_net и back_net:

```
$ brctl show br-466e46bb3d01
bridge name		bridge id		STP enabled	interfaces
br-466e46bb3d01		8000.0242a004a1eb	no		veth71aa8fb
								veth9d43fe9
Сеть back_net							vethdce3264

$ brctl show br-541b26de9489
bridge name		bridge id		STP enabled	interfaces
br-541b26de9489		8000.0242a9ed4dbf	no		veth04a8f30
								veth7d27a7f
 								veth817b4ba
```

iptables:

```
$ sudo iptables -nL -t nat
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination         
DOCKER     all  --  0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
DOCKER     all  --  0.0.0.0/0           !127.0.0.0/8          ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination         
MASQUERADE  all  --  172.19.0.0/16        0.0.0.0/0           
MASQUERADE  all  --  10.0.1.0/24          0.0.0.0/0           
MASQUERADE  all  --  10.0.2.0/24          0.0.0.0/0           
MASQUERADE  all  --  172.18.0.0/16        0.0.0.0/0           
MASQUERADE  all  --  172.17.0.0/16        0.0.0.0/0           
MASQUERADE  tcp  --  10.0.1.2             10.0.1.2             tcp dpt:9292

Chain DOCKER (2 references)
target     prot opt source               destination         
RETURN     all  --  0.0.0.0/0            0.0.0.0/0           
RETURN     all  --  0.0.0.0/0            0.0.0.0/0           
RETURN     all  --  0.0.0.0/0            0.0.0.0/0           
RETURN     all  --  0.0.0.0/0            0.0.0.0/0           
RETURN     all  --  0.0.0.0/0            0.0.0.0/0           
DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:9292 to:10.0.1.2:9292
```

Docker-proxy в процессах:

```
$ ps ax | grep docker-proxy | grep -v color
 4760 ?        Sl     0:00 /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 9292 -container-ip 10.0.1.2 -container-port 9292
```

Docker-compose:

```
$ docker-compose ps
            Name                          Command             State           Ports          
--------------------------------------------------------------------------------------------
redditmicroservices_comment_1   puma                          Up                             
redditmicroservices_post_1      python3 post_app.py           Up                             
redditmicroservices_post_db_1   docker-entrypoint.sh mongod   Up      27017/tcp              
redditmicroservices_ui_1        puma                          Up      0.0.0.0:9292->9292/tcp
```

- Имя проекта можно указать как параметром -p при запуске docker-compose, так и в файле переменных окружения .env (COMPOSE_PROJECT_NAME).
- Файл docker-compose.yml был параметризирован, примеры переменных окружений находятся в файле .env.example
- Был сочинен файл docker-compose.override.yml, использованы Volume для монтирования кода с хоста

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
