# Django site

Докеризированный сайт на Django для экспериментов с Kubernetes.

Внутри конейнера Django запускается с помощью Nginx Unit, не путать с Nginx. Сервер Nginx Unit выполняет сразу две функции: как веб-сервер он раздаёт файлы статики и медиа, а в роли сервера-приложений он запускает Python и Django. Таким образом Nginx Unit заменяет собой связку из двух сервисов Nginx и Gunicorn/uWSGI. [Подробнее про Nginx Unit](https://unit.nginx.org/).

## Деплой в Yandex.Cloud

Инструкции по деплою проекта в Yandex.Cloud смотрите [здесь](./deploy/yc-sirius/edu-berserk-jones/README.md)

## Как запустить dev-версию

Запустите базу данных и сайт:

```shell-session
$ docker-compose up
```

В новом терминале не выключая сайт запустите команды для настройки базы данных:

```shell-session
$ docker-compose run web ./manage.py migrate  # создаём/обновляем таблицы в БД
$ docker-compose run web ./manage.py createsuperuser
```

Для тонкой настройки Docker Compose используйте переменные окружения. Их названия отличаются от тех, что задаёт docker-образа, сделано это чтобы избежать конфликта имён. Внутри docker-compose.yaml настраиваются сразу несколько образов, у каждого свои переменные окружения, и поэтому их названия могут случайно пересечься. Чтобы не было конфликтов к названиям переменных окружения добавлены префиксы по названию сервиса. Список доступных переменных можно найти внутри файла [`docker-compose.yml`](./deploy/local-docker/docker-compose.yml).

## Переменные окружения

Образ с Django считывает настройки из переменных окружения:

`SECRET_KEY` -- обязательная секретная настройка Django. Это соль для генерации хэшей. Значение может быть любым, важно лишь, чтобы оно никому не было известно. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#secret-key).

`DEBUG` -- настройка Django для включения отладочного режима. Принимает значения `TRUE` или `FALSE`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#std:setting-DEBUG).

`ALLOWED_HOSTS` -- настройка Django со списком разрешённых адресов. Если запрос прилетит на другой адрес, то сайт ответит ошибкой 400. Можно перечислить несколько адресов через запятую, например `127.0.0.1,192.168.0.1,site.test`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#allowed-hosts).

`DATABASE_URL` -- адрес для подключения к базе данных PostgreSQL. Другие СУБД сайт не поддерживает. [Формат записи](https://github.com/jacobian/dj-database-url#url-schema).

## Разворачивание сайта в кластере Minikube

Запустите Minikube с драйвером VirtualBox:

```shell-session
$ minikube start --vm-driver=virtualbox
```

Активируйте плагин Ingress:
```shell-session
$ minikube addons enable ingress
```

Установите Helm (на примере установки скриптом, другие варианты [здесь](https://helm.sh/docs/intro/install/)):

```shell-session
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```

Добавьте репозиторий Bitnami в Helm:

```shell-session
$ helm repo add bitnami https://charts.bitnami.com/bitnami 
```

Установите БД Postgresql в кластер( значения в <> заменить своими):

```shell-session
$ helm install <release_name> --set auth.postgresPassword=<postgres_password>,auth.username=<username>,auth.password=<user_password>,auth.database=<db_name> bitnami/postgresql
```

Создайте .env файл с переменными окружения (значения в <> заменить указанными при установке базы данных):

```shell-session
DEBUG=False
ALLOWED_HOSTS=[<список разрешенных хостов>]
SECRET_KEY=<секретный_ключ_django>
DATABASE_URL=postgres://<username>:<user_password>@django-db-postgresql.default.svc.cluster.local:5432/<db_name>
```

С помощью `kubectl` перенесите переменные окружения в Secrets внутри кластера:

```shell-session
$ kubectl create secret generic django-secrets --from-env-file=.env
```
Из директории `backend_main_django` внутри репозитория запустить создание образа:

```shell-session
$ minikube image build -t django_app:latest .
```

Из директории `deploy/local-minikube-virtualbox` запустите сайт:

```shell-session
$ kubectl apply -f .
```

Открыть сайт можно перейдя по IP-адресу minikube. Для того, чтобы узнать IP, выполните команду

```shell-session
$ minikube ip
```

Для упрощения перехода на сайт можно задать локальный домен для данного IP-адреса. Для этого внесите адрес и домен в файл `/etc/hosts`, например:

```shell-session
192.168.59.100	star-burger.test
```

Сайт будет доступен по адресу [star-burger.test](star-burger.test).