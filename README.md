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
