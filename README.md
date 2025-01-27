# Django site

Докеризированный сайт на Django для экспериментов с Kubernetes.

Внутри конейнера Django запускается с помощью Nginx Unit, не путать с Nginx. Сервер Nginx Unit выполняет сразу две функции: как веб-сервер он раздаёт файлы статики и медиа, а в роли сервера-приложений он запускает Python и Django. Таким образом Nginx Unit заменяет собой связку из двух сервисов Nginx и Gunicorn/uWSGI. [Подробнее про Nginx Unit](https://unit.nginx.org/).

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

Для тонкой настройки Docker Compose используйте переменные окружения. Их названия отличаются от тех, что задаёт docker-образа, сделано это чтобы избежать конфликта имён. Внутри docker-compose.yaml настраиваются сразу несколько образов, у каждого свои переменные окружения, и поэтому их названия могут случайно пересечься. Чтобы не было конфликтов к названиям переменных окружения добавлены префиксы по названию сервиса. Список доступных переменных можно найти внутри файла [`docker-compose.yml`](./docker-compose.yml).

## Запуск в кластере Minikube

Установите minikube в virtualbox:
```
minikube start --driver=virtualbox
```
Запустите ingress:
```
minikube addons enable ingress
```
Часть настроек берётся из переменных окружения. Чтобы их определить, откройте файл ConfigMap.yaml и запишите туда данные в таком формате: ПЕРЕМЕННАЯ=значение.
`SECRET_KEY`
`DEBUG`
`ALLOWED_HOSTS`
`DATABASE_URL`

Установите [Helm](https://helm.sh/)

Добавьте Bitnami
```
helm repo add bitnami https://charts.bitnami.com/bitnami
```
Установите PostgreSQL:
```
helm install my-postgresql bitnami/postgresql --set postgresqlPassword=password,postgresqlUsername=myuser
```
Запустите манифесты:
```
kubectl apply -f django-app.yaml
```
```
kubectl apply -f ingress.yaml
```
```
kubectl apply -f django-clearsessions.yaml
```

## Запуск в облаке

Рабочая версия сайта [k8s-test-django](https://edu-dreamy-goodall.sirius-k8s.dvmn.org/admin/login/?next=/admin/)

Часть настроек берётся из переменных окружения. Чтобы их определить, откройте файл ConfigMap.yaml и запишите туда данные в таком формате: ПЕРЕМЕННАЯ=значение.
`SECRET_KEY`
`DEBUG`
`ALLOWED_HOSTS`
`DATABASE_URL`

Установите [Helm](https://helm.sh/)

Добавьте Bitnami
```
helm repo add bitnami https://charts.bitnami.com/bitnami
```
Установите PostgreSQL:
```
helm install my-postgresql bitnami/postgresql --namespace <your_namespace>
```
Заполните манифест файл ingress-prod.yaml:
- host: your.host.ru

Заполните манифест файл django-app-prod.yaml:
- nodePort: your_nodePort

Запустите манифесты:
```
kubectl apply -f ConfigMap-prod.yaml
```
```
kubectl apply -f django-app-prod.yaml
```
```
kubectl apply -f ingress-prod.yaml
```
```
kubectl apply -f django-clearsessions-prod.yaml
```

## Переменные окружения

Образ с Django считывает настройки из переменных окружения:

`SECRET_KEY` -- обязательная секретная настройка Django. Это соль для генерации хэшей. Значение может быть любым, важно лишь, чтобы оно никому не было известно. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#secret-key).

`DEBUG` -- настройка Django для включения отладочного режима. Принимает значения `TRUE` или `FALSE`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#std:setting-DEBUG).

`ALLOWED_HOSTS` -- настройка Django со списком разрешённых адресов. Если запрос прилетит на другой адрес, то сайт ответит ошибкой 400. Можно перечислить несколько адресов через запятую, например `127.0.0.1,192.168.0.1,site.test`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#allowed-hosts).

`DATABASE_URL` -- адрес для подключения к базе данных PostgreSQL. Другие СУБД сайт не поддерживает. [Формат записи](https://github.com/jacobian/dj-database-url#url-schema).
