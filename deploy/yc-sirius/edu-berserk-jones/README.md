## Развертывание проекта в кластере Yandex.Cloud

- Устанавливаем утилиту `kubectl`. Указанный ниже способ подходит для Arch-based дистрибутивов. Остальные способы [здесь](https://kubernetes.io/ru/docs/tasks/tools/install-kubectl/).

```bash
sudo pacman -S kubectl
```

- Устанавливаем `Yandex.Cloud CLI`:

```bash
curl -sSL https://storage.yandexcloud.net/yandexcloud-yc/install.sh | bash
```

- Получаем токен [Yandex ID](https://oauth.yandex.ru/authorize?response_type=token&client_id=1a6990aa636648e9b2ef855fa7bec2fb)

- Инициализируем Yandex.Cloud CLI:

```bash
yc init
```

В процессе инициализации утилита запросит токен, полученный на втором шаге.

- Настраиваем `kubectl` на работу с Яндекс облаком.

```bash
yc managed-kubernetes cluster get-credentials --id <идентификатор_кластера> --external
```

- Выбираем пространство имен для работы, либо явно указываем `namespace` в конфигах.

```bash
kubectl config set-context --current --namespace=<insert-namespace-name-here>
```

- Создаем .env с необходимыми параметрами:

```bash
SECRET_KEY=<секретный ключ Django>
DATABASE_URL=postgres://<username>:<password>@<host>:<port>/<DB name>
DEBUG=False
DJANGO_SUPERUSER_USERNAME=<логин суперпользователя>
DJANGO_SUPERUSER_PASSWORD=<пароль суперпользователя>
DJANGO_SUPERUSER_EMAIL=<email суперпользователя>
```

- Создаем секрет из файла:

```bash
$ kubectl create secret generic <имя секрета> --from-env-file=.env
```

- Скачиваем сертификат Яндекс.Облака и заворачиваем его в секрет k8s для подгрузки к подам:

```bash
wget "https://storage.yandexcloud.net/cloud-certs/CA.pem" \
--output-document root.crt && \
chmod 0600 root.crt && \
kubectl create secret generic <имя_секрета> --from-file=root.crt
```

- Запускаем проект:

```bash
kubectl apply -f .
```