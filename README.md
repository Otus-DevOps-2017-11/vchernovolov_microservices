# vchernovolov_microservices

## ДЗ-14 "Технология контейнеризации. Введение в Docker"

### Установлен docker
Проверка версии
```
docker version
```
> Client:<br>
  &nbsp;&nbsp;&nbsp;Version:	17.12.0-ce<br>
  &nbsp;&nbsp;&nbsp;API version:	1.35<br>
  &nbsp;&nbsp;&nbsp;Go version:	go1.9.2<br>
  &nbsp;&nbsp;&nbsp;Git commit:	c97c6d6<br>
  &nbsp;&nbsp;&nbsp;Built:	Wed Dec 27 20:11:19 2017<br>
  &nbsp;&nbsp;&nbsp;OS/Arch:	linux/amd64<br>

> Server:<br>
  &nbsp;&nbsp;&nbsp;Engine:<br>
  &nbsp;&nbsp;&nbsp;Version:	17.12.0-ce<br>
  &nbsp;&nbsp;&nbsp;API version:	1.35 (minimum version 1.12)<br>
  &nbsp;&nbsp;&nbsp;Go version:	go1.9.2<br>
  &nbsp;&nbsp;&nbsp;Git commit:	c97c6d6<br>
  &nbsp;&nbsp;&nbsp;Built:	Wed Dec 27 20:09:53 2017<br>
  &nbsp;&nbsp;&nbsp;nux/amd64<br>
  &nbsp;&nbsp;&nbsp;Experimental:	false<br>



### Список команд, использованных в ходе выполнения ДЗ
```
docker run hello-world
docker ps
docker ps -a
docker images
docker run <image_name> [--rm]
  docker run -it ubuntu:16.04 /bin/bash
docker ps -a --format "table {{.ID}}\t{{.Image}}\t{{.CreatedAt}}\t{{.Names}}"
docker start <u_container_id>
docker attach <u_container_id>
docker exec -it <u_container_id> <cmd>
docker commit <u_container_id> <image_name>
docker exec -it <u_container_id> <cmd>
docker commit <u_container_id> <image_name> # create image
docker kill <u_container_id>
docker stop <u_container_id>
docker system df
docker rm <u_container_id> [-f]
  docker rm $(docker ps -a -q)
docker rmi <u_image_id>
  docker rmi $(docker images -q)
```


## ДЗ-14 "Docker-контейнеры"

### Установлен компонент ```docker-machine```
```
curl -L https://github.com/docker/machine/releases/download/v0.13.0/docker-machine-`uname -s`-`uname -m` >/tmp/docker-machine && \
sudo install /tmp/docker-machine /usr/local/bin/docker-machine
```

### Сконфигурирован ```google-cloud```
Создан новый проект в ```GCE```<br>
Инициализирован ```google-cloud```
```
gcloud init
```

### Создан хост ```docker-host```
```
docker-machine create --driver google \
  --google-project <project_id> \
  --google-zone europe-west1-b \
  --google-machine-type g1-small \
  --google-machine-image $(gcloud compute images list --filter ubuntu-1604-lts --uri) \
  docker-host
```
Проверка успешности создания хоста
```
docker-machine ls
```

### Созданы файлы конфигурации, запуска образа с приложением
- Dockerfile - описание образа
- db_config - конфиг для mongodb
- mongod.conf - переменная со ссылкой mongodb
- start.sh - скрипт запуска приложения


### Созбран образ
```
docker build -t reddit:latest .
```
Проверка результата
```
docker images -a
```

### Запущен контейнер
```
docker run --name reddit -d --network=host reddit:latest
```
Проверка результата
```
docker-machine ls
```

### Создано правило firefall, разрешающее входящий трафик на порт 9292
```
gcloud compute firewall-rules create reddit-app \
  --allow tcp:9292 \
  --priority=65534 \
  --target-tags=docker-machine \
  --description="Allow TCP connections" \
  --direction=INGRESS
```

### Создана учетка, залит образ в репозиторий ```Docker hub```
```
docker tag reddit:latest <login-on-docker-hub>/otus-reddit:1.0
```
```
docker push <login-on-docker-hub>/otus-reddit:1.0
```

### * Namespaces
```
docker run --rm -ti tehbilly/htop
```
```
docker run --rm --pid host -ti tehbilly/htop
```
В первом случае ведем мониторинг процессов внутри docker контейнера<br>
Во втором - мониторинг на хосте docker контейнера


## ДЗ-16 "Docker-образа. Микросервисы"

### Приложение разбито на несколько компонент
Прошлые наработки вынесены в директорию docker-monolith.<br><br>
В директории reddit-microservices размещено новое приложение (компоненты приложения).<br>
Компоненты:
- post-py - сервис, отвечающий за написание постов;
- comment - сервис, отвечающий за написание комментариев;
- ui - веб-интерфейс, работающий с другими сервисами.

### Сконфигурированы файлы ```Dockerfile``` каждого компонента приложения

### Собрано приложение
Установлен последний образ ```mongodb```
```
docker pull mongo:latest
```
Собраны образа сервисов (компонентов приложения):
```
docker build -t <dockerhub-login>/post:1.0 ./post-py
docker build -t <dockerhub-login>/comment:1.0 ./comment
docker build -t <dockerhub-login>/ui:1.0 ./ui
```

### Проверена работа приложения
Создана сеть для приложения
```
docker network create reddit
```

Запущены контейнеры
```
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db mongo:latest
docker run -d --network=reddit --network-alias=post <dockerhub-login>/post:1.0
docker run -d --network=reddit --network-alias=comment <dockerhub-login>/comment:1.0
docker run -d --network=reddit -p 9292:9292 <dockerhub-login>/ui:1.0
```

### Создан ```volume``` для ```mongodb```
Перезапущено приложение (контейнеры), для ```mongo``` указан созданный ```volume```
```
docker kill $(docker ps -q)
docker run -d --network=reddit —network-alias=post_db \
 --network-alias=comment_db -v reddit_db:/data/db mongo:latest
docker run ...
```
После перезапуска приложения добавленные ранее посты остаются.

## ДЗ-17 "Docker: сети, docker-compose"

### Запущены контейнеры с использованием различных ```network```-драйверов

* Драйвер ```none```
```
docker run --network none --rm -d --name net_test joffotron/docker-net-tools -c "sleep 100"
```
* Драйвер ```host```
```
docker run --network host --rm -d --name net_test joffotron/docker-net-tools -c "sleep 100"
```

* Драйвер ```bridge```<br>
Создана сеть
```
docker network create reddit --driver bridge
```
Запущены контейнеры
```
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db mongo:latest
docker run -d --network=reddit --network-alias=post <user-login>/post:1.0
docker run -d --network=reddit --network-alias=comment <user-login>/comment:1.0
docker run -d --network=reddit -p 9292:9292 <user-login>/ui:2.0
```
Созданы сети ```front_net```, ```back_net```
```
docker network create back_net --subnet=10.0.2.0/24
docker network create front_net --subnet=10.0.1.0/24
```
Запущены контейнеры в разделенных сетях ```front_net```, ```back_net```
```
docker run -d --network=front_net -p 9292:9292 --name ui  <user-login>/ui:2.0
docker run -d --network=back_net --name comment <user-login>/comment:1.0
docker run -d --network=back_net --name post <user-login>/post:1.0
docker run -d --network=back_net --name mongo_db --network-alias=post_db --network-alias=comment_db mongo:latest
```
Подключены контейнеры ко второй сети
```
docker network connect front_net post
docker network connect front_net comment
```

### Установлен ```docker-compose```

### Реализован файл ```docker-compose.yml```
Параметры ```docker-compose.yml``` определены посредством переменных окружения, заданных в файле ```.env```<br>
Запуск контейнеров
```
docker-compose up -d
```


### Задание ```docker-compose *```

Чтобы изменить имя проекта по-умолчанию, задаем в .env переменную
```
COMPOSE_PROJECT_NAME=project_name
```
