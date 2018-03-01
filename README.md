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
