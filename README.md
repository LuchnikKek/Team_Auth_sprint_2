https://github.com/Ezereul/Auth_sprint_2

# Backend онлайн-кинотеатра

Backend онлайн-кинотеатра с микросервисной архитектурой.

- Сервис авторизации
- Сервис выдачи контента
- Сервис admin-панель
- Сервис ETL

Поднимается с помощью единого Docker compose.

## Quick setup

Перед первым запуском нужно подготовить переменные окружения.

В директории с проектом запустить:

```shell
$ mv .env.template .env
$ openssl genrsa -out rsa.key 2048
$ openssl rsa -in rsa.key -pubout > rsa.key.pub
```
Затем открыть файл `.env` в любом текстовом редакторе. 
В качестве значения переменной `AUTHJWT_PRIVATE_KEY` установить значение из файла `rsa.key`.
В качестве значения переменной `AUTHJWT_PUBLIC_KEY` установить значение из файла `rsa.key.pub`.

(Желательно) В том же `.env` изменить значение `DJANGO_SECRET_KEY` на своё. 

## О сервисах

### Сервис авторизации

Endpoint документации: `/api/v1/auth/openapi`.

Написан на FastAPI. Служит единой точкой входа.

- Регистрация и аутентификация
- OAuth 2.0 с авторизацией по Яндекс ID
- Выдача токенов с асимметричным шифрованием
- Кэширование refresh-токенов в Redis
- CRUD для ролей. Назначение, лишение пользователя.
- История входов. Партицирована по месяцам. На ручке выдачи соответствующая пагинация.
- Rate Limit со скользящим окном для решения проблемы длительного блока пользователя при отправке запросов на стыке минут.
- Трасировка запросов
- Создание базовых ролей, суперпользователя. Логин: `admin`, пароль: `11111`.
- Основные эндпоинты покрыты асинхронными тестами 

Код находится в директории `./auth`.


### Сервис выдачи контента

Endpoint документации: `/api/v1/movies/openapi`.

Написан на FastAPI. Служит GET-api для получения данных из Elasticsearch.

- Доступ есть только у авторизованных пользователей.
- Сортировка, фильтрация, пагинация, нечёткий поиск.
- Кэширование всех запросов в Redis.
- Все эндпоинты покрыты асинхронными тестами.

Код находится в директории `./movies_api`. Больше информации в Swagger-документации.


### Сервис admin-панель

Admin-панель на Django для редактирования контента, хранящегося в PostgreSQL.

Логин, пароль по умолчанию: `admin:11111`.

Настроен Custom backend для перенаправления авторизации в сервис авторизации. В остальном обычная Django-админка.

Код находится в директории `./movies_admin`.


### Сервис ETL

ETL-сервис для переноса обновлённого контента из PostgreSQL в Elasticsearch.

[Более подробный README.](movies_etl/README.md)

Код находится в директории `./movies_etl`.

### Moreover

- __Nginx__ в качестве веб-сервера.
- Единый `docker-compose` и `.env` для запуска.
- Свой `Dockerfile` у каждого сервиса.
- __Poetry__ в качестве пакетного менеджера.
- Датасет прилагается, расположен в `./data`.
