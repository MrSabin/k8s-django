## Развертывание проекта в кластере Yandex.Cloud

1. Устанавливаем утилиту `kubectl`. Указанный ниже способ подходит для Arch-based дистрибутивов. Остальные способы [здесь](https://kubernetes.io/ru/docs/tasks/tools/install-kubectl/).

```bash
sudo pacman -S kubectl
```

2. Устанавливаем `Yandex.Cloud CLI`:

```bash
curl -sSL https://storage.yandexcloud.net/yandexcloud-yc/install.sh | bash
```

3. Получаем токен [Yandex ID](https://oauth.yandex.ru/authorize?response_type=token&client_id=1a6990aa636648e9b2ef855fa7bec2fb)

4. Инициализируем Yandex.Cloud CLI:

```bash
yc init
```

В процессе инициализации утилита запросит токен, полученный на втором шаге.

5. Настраиваем `kubectl` на работу с Яндекс облаком.

```bash
yc managed-kubernetes cluster get-credentials --id <идентификатор_кластера> --external
```

6. Остается лишь выбрать пространство имен для работы, либо явно указывать `namespace` в конфигах.

```bash
kubectl config set-context --current --namespace=<insert-namespace-name-here>
```

7. Запускаем проект:

```bash
kubectl apply -f .
```