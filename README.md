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


## ДЗ-19 "Gitlab CI. Построение процесса непрерывной интеграции"

### Создана виртуальная машина в Google Cloud, установлен docker
В ```Google Cloud``` создана виртуальная машина gitlab-ci<br>
На данной машине установлен ```docker```, ```docker-compose```.
```
# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
# add-apt-repository "deb https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
# apt-get update
# apt-get install docker-ce docker-compose
```

### На виртуальной машине установлен ```Gitlab```
Модифицирован файл ```docker-compose.yml```, для запуска ```Gitlab``` использована ```omnibus```-установка<br>
Поднят ```Gitlab```

```
docker-compose up -d
```
Проверена работа ```Gitlab``` - переходя по адресу
```
http://<gitlab-ci-vm-ip>
```
отобразился интерфейс ```Gitlab```

### Произведены настройки ```Gitlab```
- Отключена регистрация новых пользователей
- Создана группа проектов
- Создан проект

### Определен ```CD/CI Pipeline``` проекта
В репозиторий добавлен файл ```.gitlab-ci.yml``` с определением этапов ```Pipeline```<br>
Определены ```Jobs``` для этапов.

### Настроен ```Runner```
На витуальной машине gitlab-ci запущен, зарегистрирован ```Runner```
```
docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest
```
```
docker exec -it gitlab-runner gitlab-runner register
```

### Исходный код приложения ```Reddit``` добавлен в репозиторий
```
git clone https://github.com/express42/reddit.git && rm -rf ./reddit/.git
git add reddit/
git commit -m “Add reddit app”
git push gitlab docker-6
```

### Тестирование приложения ```Reddit```
В репозиторий добавлен файл ```simpletest.rb``` - вызов теста ```Reddit```<br>
Файл ```.gitlab-ci.yml``` модифицирован (+ добавлен файл ```simpletest.rb``` на этапе ```test_unit_job```)<br>
В файл ```reddit/Gemfile``` добавлена библиотека для тестирования
```
...
gem 'rack-test'
...
```
Проверена работа запуска, работы теста на каждое изменение в коде.


## ДЗ-20 "Устройство Gitlab CI. Непрерывная поставка"

### Создан новый проект для расширения ```Pipeline```
После запуска vm (остановлена с предыдущего ДЗ), скорректирован файл ```docker-compose.yml```, указан новый vm-ip, перезапущен контейнер ```gitlab/gitlab-ce```
```
  ...
  GITLAB_OMNIBUS_CONFIG: |
        external_url 'new-vm-ip'
  ...
```

### Включен ранее зарегистрированный ```runner``` для нового проекта

### Определены различные окружения CD/CI, условия/ограничения, динамические окружения
В файле ```gitlab-ci.yml``` определены окружения Dev, Staging, Production<br>
```
stages:
...
  - stage
  - production

...
deploy_dev_job:
  stage: review
  script:
    - echo 'Deploy'
  environment:
    name: dev
    url: http://dev.example.com

...
staging:
  stage: stage
  when: manual
  script:
    - echo 'Deploy'
  environment:
    name: stage
    url: https://beta.example.com

production:
  stage: production
  when: manual
  script:
    - echo 'Deploy'
  environment:
    name: production
    url: https://example.com
```
Далее заданы ограничения для окружений Staging, Production - ```semver (Semantic Versioning)``` тег ```git```
```
...
staging:
  stage: stage
  ...
  only:
    - /^\d+\.\d+.\d+/
  ...
```
Изменения без указания тэга запустят ```Pipeline``` без job staging  и production<br>
Определены динамические окружения для каждой feature-ветки (кроме ```master```)
```
branch review:
  stage: review
  script: echo "Deploy to $CI_ENVIRONMENT_SLUG"
  environment:
    name: branch/$CI_COMMIT_REF_NAME
    url: http://$CI_ENVIRONMENT_SLUG.example.com
  only:
    - branches
  except:
    - master
```

## ДЗ-21 "Введение в мониторинг. Системы мониторинга"

### Установлен, запущен ```Prometheus```
Созданы правила для ```Prometheus``` & ```Puma```

```
gcloud compute firewall-rules create prometheus-default --allow tcp:9090
gcloud compute firewall-rules create puma-default --allow tcp:9292
```

Создан ```Docker``` хост в ```GCE``` и настроено локальное окружение для работы с ним
```
export GOOGLE_PROJECT=your_project

docker-machine create --driver google \
  --google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
  --google-machine-type n1-standard-1 vm1

eval $(docker-machine env vm1)

docker run --rm -p 9090:9090 -d --name prometheus prom/prometheus

```

### Произведена реорганизация структуры каталогов
- перенесено содержимое каталога ```reddit-microservices``` в корень репозитория (каталог ```reddit-microservices``` удален);
- в корне репозитория создан каталог ```src```, в него перенесены компоненты приложения ```reddit```: ```comment```, ```post-py```, ```ui```;
- создан каталог ```docker``` в корне репозитория, в него перенесен каталог ```docker-monolith``` и файлы ```docker-compose``` и все ```.env```;
- в корне репозитория создан каталог ```monitoring``` (там складываем все, относящееся к мониторингу);

### Создан ```Docker``` образ ```Prometheus```, сконфигурирован ```Prometheus```
```
export USER_NAME=username
docker build -t $USER_NAME/prometheus .
```

### Собраны образы компонентов приложения ```reddit```
Запуск из корня репозитория:
```
for i in ui post-py comment; do cd src/$i;
bash docker_build.sh; cd -; done
```

### В ```docker-compose.yml``` определен сервис - ```Prometheus```
Подняты все сервисы, определенные в ```docker-compose.yml```
```
docker-compose up -d
```

### Проверена работа мониторинга состояния микросервисов
Проверка индикаторов **\*health\*** <br>
Остановлен сервис `post`
```
docker-compose stop post
```
Проверка значения индикартора **ui_health_post_availability** - получили значение 0
Запущен сервис `post`
```
docker-compose start post
```
После запуска - значение индикатора **ui_health_post_availability** = 1

### Настроен сбор метрик хоста с помощью `Node exporter`
- в `docker/docker-compose.yml` добавлен сервис - `node-exporter`;
- в `monitoring/prometheus/prometheus.yml` добавлен еще один job - `'node'`
- перезапущены сервисы:
```
docker-compose down
docker-compose up -d
```

### Собранные образы запушены на `dockerhub`
```
docker push $USER_NAME/ui
docker push $USER_NAME/comment
docker push $USER_NAME/post
docker push $USER_NAME/prometheus
```
Ссылка на ```dockerhub``` образы:
```
https://hub.docker.com/u/vchgacc1/
```

## ДЗ-23 "Мониторинг приложения и инфраструктуры"

### Создан Docker хост в GCE и настроено локальное окружение на работу с ним
Создан хост, аналогично пред. ДЗ, собраны образы компонентов приложения `reddit`, `prometheus`

### Разделены файлы `docker-compose`
В файле `docker-compose.yml` оставлено только описание компонентов приложения `reddit`<br>
Описание компонентов, относящихся к мониторингу, вынесено в файл `docker-compose-monitoring.yml`

### Установлен `cAdvisor`
Для мониторинга состояния контейнеров<br>
Новый сервис добавлен в файл `docker-compose-monitoring.yml`<br>
Добавлена инф. в `prometheus.yml` (для начала сбора метрик `Prometheus`)

### Установлен инструмент `Grafana` для визуализации данных из `Prometheus`
Новый сервис добавлен в файл `docker-compose-monitoring.yml`<br><br>
Создан `data source` с типом - `Prometheus`<br><br>
Импортирован дашбоард *DockerMonitoring* (источник - https://grafana.com/dashboards)<br><br>
Добавлены дашбоарды:
- *UI_Service_Monitoring*;
- *Business_Logic_Monitoring*;

### Добавлен `Alertmanager`
Компонент для `Prometheus`, обрабатывающий алерты и отправляющий оповещения по заданному назначению<br><br>
В качестве источника для отправки сообщений указан Slack-канал (`slack-notifications channel` - `Incoming Webhook`, созданный в рамках ДЗ)<br><br>
Собран образ `alertmanager`
```
docker build -t $USER_NAME/alertmanager .
```
Новый сервис добавлен в файл `docker-compose-monitoring.yml`<br><br>
Определено правило алерта (`alert.yml`)<br><br>
Добавлена информация о правилах в конфиг `Prometheus`<br><br>
Пересобран образ `Prometheus`
```
docker-build -t $USER_NAME/prometheus .
```
Проверена работа алерта - остановлен один из сервисов:
```
docker-compose stop post
```
Через некоторое время в канал для нотификаций алертинга пришло сообщение *InstanceDown* (недоступность сервиса)

### Собранные образы запушены на `DockerHub`
```
docker login
docker push $USER_NAME/ui
docker push $USER_NAME/comment
docker push $USER_NAME/post
docker push $USER_NAME/prometheus
docker push $USER_NAME/alertmanager
```

## ДЗ-25 "Логирование и распределенная трассировка"

### Обновлен код приложения `reddit`
Обновлен код приложения `reddit` (https://github.com/express42/reddit/tree/logging)<br>
Собраны образы компонентов приложения<br>
Запуск из корня репозитория:
```
for i in ui post-py comment; do cd src/$i;
bash docker_build.sh; cd -; done
```
Запушены опразы на DockerHub<br>

### Создан Docker хост в GCE и настроено локальное окружение на работу с ним
```
export GOOGLE_PROJECT=google_project_id

docker-machine create --driver google \
  --google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
  --google-machine-type n1-standard-1 \
  --google-open-port 5601/tcp \
  --google-open-port 9292/tcp \
  --google-open-port 9411/tcp \
  logging

eval $(docker-machine env logging)

```

### Создан отдельный compose-файл для системы логирования
Настроены сервисы `fluentd`, `elasticsearch`, `kibana`<br>
Оказалось, для работы сервиса post, нужно сразу включить в файл сервис `zipkin`..<br>
Конфигурация `fluentd` определена в файле `fluent.conf`

### Определен драйвер логирования `fluentd` для сервисов `post`, `ui`

### Заданы фильтры парсинга логов (конфиг `fluentd`)
