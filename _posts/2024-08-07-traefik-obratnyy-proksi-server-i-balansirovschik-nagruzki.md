---
layout: post
title: Traefik v3.0 + Docker. Почти подробное руководство.
description: Рассматривается вариант установи и настройки современного, динамического обратного прокси  Traefik.
image: X97Mi.jpg
categories:
- DevOps
- Traefik
tags:
- traefik
- devops
- linux
date: 2024-08-07 17:58 +0300
---
## Немного теории.

### Основы — Что такое обратный прокси?

Обратный прокси-сервер (Reverse Proxy) — это тип сервера, который принимает запросы от клиентов, передает их на один или несколько серверов и возвращает ответы клиентам от имени этих серверов. Обратный прокси-сервер располагается перед одним или несколькими веб-серверами и выполняет различные функции, такие как распределение нагрузки, кэширование, шифрование, сжатие и обеспечение безопасности.

### Основные функции обратного прокси-сервера:

1. Распределение нагрузки: Обратный прокси может распределять входящий трафик между несколькими серверами, что позволяет равномерно распределять нагрузку и предотвращать перегрузку одного сервера.

2. Кэширование: Прокси-сервер может сохранять копии часто запрашиваемого контента, чтобы ускорить обработку последующих запросов и уменьшить нагрузку на исходные серверы.

3. Шифрование: Обратный прокси может обрабатывать SSL/TLS шифрование и дешифрование, разгружая веб-серверы от этой задачи и улучшая производительность.

4. Безопасность: Прокси-сервер может фильтровать вредоносный трафик, защищая внутренние серверы от атак. Также он может скрывать IP-адреса внутренних серверов, обеспечивая дополнительный уровень безопасности.

5. Сжатие: Обратный прокси может сжимать ответы от серверов перед отправкой их клиентам, что снижает объем передаваемых данных и ускоряет загрузку страниц.

6. Реализация политик доступа: Прокси-сервер может ограничивать доступ к определенным ресурсам или серверам на основе различных критериев, таких как IP-адреса клиентов, время суток и другие параметры.

7. Аутентификация и авторизация: Прокси-сервер может управлять аутентификацией и авторизацией пользователей перед передачей запросов на конечные серверы. Это может включать проверку учетных данных, токенов доступа и другие механизмы безопасности.

### Схема работы обратного прокси-сервера:

```yaml
Клиент -> Обратный прокси-сервер -> Веб-серверы
            ^                  ^
            |                  |
       Прием запросов      Отправка ответов
```
Обратный прокси-сервер принимает запросы от клиентов и решает, на какой из внутренних серверов передать запрос. После получения ответа от веб-сервера, прокси-сервер отправляет ответ клиенту.

### Принцип работы:

1. Прием запросов: Обратный прокси-сервер получает HTTP(S) запросы от клиентов (например, браузеров).

2. Переадресация запросов: После получения запроса прокси-сервер анализирует его и перенаправляет на соответствующий сервер в своей внутренней сети.

3. Получение ответа: Обратный прокси-сервер получает ответ от конечного сервера.

4. Отправка ответа клиенту: Прокси-сервер отправляет ответ клиенту, как если бы он сам являлся конечным сервером.

## Минимальная конфигурация для запуска Traefik

Установить Traefik можно разными способами: от запуска бинарного файла, до использования Helm Chart в Kubernetes. В рамках данной статьи, будем рассматривать способ установки при помощи Docker и управлять через Docker Compose. 
<!-- Что это такое и как установить читаем: Установка Docker и Docker Compose. -->


Минимальный docker-compose.yml выглядит так:

```yaml
services:
  traefik:
    image: traefik:latest
    ports:
      - 8080:8080
    command: "--api=true --api.dashboard=true --api.insecure=true"
```
{: file='docker-compose.yml'}

Далее сохраняем docker-compose.yml и запускаем его:

``` bash
sudo docker compose up -d
```
Информационная панель будет доступна по адресу <http://127.0.0.1:8080/dashboard/#/>.

## Назначение доменного имени

Обращаться к сервису по ip-адресу и порту технически возможно, но очень не удобно, к тому же не безопасно. Доменное имя вида **traefik.mylab.oakazanin.ru** запоминать и использовать будет значительно удобнее.

Для того чтобы все компьютеры в сети могли обращаться к Traefik по доменному имени, необходимо настраивать службу доменных имен, но об этом позже. Пока предоставим доступ к системной панели Traefik по доменному имени **traefik.mylab.oakazanin.ru** только в границах локального компьютера, на котором запущена служба.

Для этого добавим в файл:

```bash
sudo nano /etc/hosts
```
```bash
127.0.0.1 traefik.mylab.oakazanin.ru  traefik
```
а docker-compose.yml приведем к виду:

```yaml
services:
  traefik:
    image: traefik:latest
    ports:
      - 8080:8080
      - 80:80
    command: "--api=true --api.dashboard=true --api.insecure=true --entrypoints.http.address=:80 --providers.docker=true --providers.docker.endpoint=unix:///var/run/docker.sock"
    labels:
      - "traefik.http.routers.traefik-dashboard.entrypoints=http"
      - "traefik.http.routers.traefik-dashboard.rule=Host(`traefik.mylab.oakazanin.ru`)"
      - "traefik.http.routers.traefik-dashboard.service=dashboard@internal"
      - "traefik.http.routers.traefik-dashboard-api.entrypoints=http"
      - "traefik.http.routers.traefik-dashboard-api.rule=Host(`traefik.mylab.oakazanin.ru`) && PathPrefix(`/api`)"
      - "traefik.http.routers.traefik-dashboard-api.service=api@internal"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
```
{: file='docker-compose.yml'}
И перезапустим контейнер

```bash
sudo docker compose up -d
```
```bash
PING traefik.mylab.oakazanin.ru (127.0.0.1) 56(84) bytes of data.
64 bytes from localhost (127.0.0.1): icmp_seq=1 ttl=64 time=0.065 ms
64 bytes from localhost (127.0.0.1): icmp_seq=2 ttl=64 time=0.068 ms
64 bytes from localhost (127.0.0.1): icmp_seq=3 ttl=64 time=0.081 ms
64 bytes from localhost (127.0.0.1): icmp_seq=4 ttl=64 time=0.170 ms
64 bytes from localhost (127.0.0.1): icmp_seq=5 ttl=64 time=0.059 ms
```
В приведенной выше конфигурации (docker-compose.yml) сначала опубликовали порт 80:80, который используется веб-браузером по умолчанию для http подключений и добавили `--entrypoints.http.address=:80` в параметр запуска Traefik, тем самым создав точку входа в сеть под названием `http`.
Далее, в раздел `volumes` добавили `/var/run/docker.sock:/var/run/docker.sock:ro` чтобы Traefik смог прослушивать события Docker API, а в параметрах запуска `--providers.docker=true --providers.docker.endpoint=unix:///var/run/docker.sock` определяем подключение к Unix сокету для обмена данными между процессами на хосте.

> Запись вида 80:80 обозначает привязку порта 80 контейнера к порту 80 хоста, где слева порт хоста, а справа порт контейнера.
{: .prompt-info }

Затем создаем маршрут, в котором указываем, что сервис `dashboard` будет доступен по адресу `traefik.mylab.oakazanin.ru` используя `http` протокол. А также сервис `api`, доступный по адресу `traefik.mylab.oakazanin.ru` с префиксом `/api` используя всё тот же `http` протокол.
Записи `dashboard@internal` и `api@internal` являются псевдонимами внутренних сервисов Traefik, которые равносильны записи `имя контейнера@ip-контейнера:порт контейнера`.

### Убираем лишние порты для внешних подключений

В приведенной конфигурации, мы можем получить доступ к одной и той же службе извне через порт `80` и `8080`. Согласно документации, Traefik будет использовать порт, который меньше, т.е. `80`, таким образом становится бессмысленным пробрасывать наружу порт `8080`.
Следовательно, запись вида:
```yaml
ports:
  - 8080:8080
  - 80:80
```
{: file='docker-compose.yml'}

скорректируем

```yaml
ports:
  - 80:80
```
{: file='docker-compose.yml'}

### Оптимизируем конфигурационный файл

В приведенной выше конфигурации командная строка, в которой мы управляем Traefik, выглядит следующим образом: 

```yaml
command: "--api=true --api.dashboard=true --api.insecure=true --entrypoints.http.address=:80 --providers.docker=true --providers.docker.endpoint=unix:///var/run/docker.sock"
```
{: file='docker-compose.yml'}

На практике, количество параметров может быть более двадцати, и если использовать вышеуказанный стиль написания, то в дальнейшем его будет очень сложно настраивать и отлаживать. Приведем его в более "удобочитаемы" вариант.

```yaml
command:
  - "--api=true"
  - "--api.dashboard=true"
  - "--api.insecure=true"
  - "--entrypoints.http.address=:80"
  - "--providers.docker=true"
  - "--providers.docker.endpoint=unix:///var/run/docker.sock"
```
{: file='docker-compose.yml'}

### Добавляем проверку работоспособности для сервисов

Все сервисы рано или поздно могут давать сбой. Наша задача своевременно выявить проблему и принять необходимые меры для ее устранения. В этом нам может помочь встроенная в Traefik служба `healthcheck`, которая сигнализирует нам если служба перестала отвечать на запросы. Реализуется это переодической отправкой `ping` запроса, который должен возвращать 0 если сервис здоров и 1 если сервис не исправен.
Для активации службы необходимо прописать в раздел `command`
```yaml
--ping=true
```
{: file='docker-compose.yml'}

И добавить в `docker-compose.yml` раздел самопроверки:

```yaml
healthcheck:
  test: ["CMD-SHELL", "wget -q --spider --proxy off localhost:8080/ping || exit 1"]
  interval: 3s
  retries: 10
```
{: file='docker-compose.yml'}

Данный блок кода выполняет самопроверку, который выполняется каждые 3 секунды и сообщает Docker, что состояние сервиса "не здоров" после 10 последовательно неудачных опросов.
   
Чтобы служба могла автоматически перезапускаться при возникновении проблем, мы можем добавить в конфигурацию следующее:
```yaml
restart: always
```
{: file='docker-compose.yml'}

В финальной версии файл `docker-compose.yml` выглядит следующим образом:
```yaml
services:
  traefik:
    image: traefik:latest
    restart: always
    ports:
      - 80:80
    command:
      - "--api=true"
      - "--api.dashboard=true"
      - "--api.insecure=true"
      - "--ping=true"
      - "--entrypoints.http.address=:80"
      - "--providers.docker=true"
      - "--providers.docker.endpoint=unix:///var/run/docker.sock"
    labels:
      - "traefik.http.routers.traefik-dashboard.entrypoints=http"
      - "traefik.http.routers.traefik-dashboard.rule=Host(`traefik.mylab.oakazanin.ru`)"
      - "traefik.http.routers.traefik-dashboard.service=dashboard@internal"
      - "traefik.http.routers.traefik-dashboard-api.entrypoints=http"
      - "traefik.http.routers.traefik-dashboard-api.rule=Host(`traefik.mylab.oakazanin.ru`) && PathPrefix(`/api`)"
      - "traefik.http.routers.traefik-dashboard-api.service=api@internal"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    healthcheck:
      test: ["CMD-SHELL", "wget -q --spider --proxy off localhost:8080/ping || exit 1"]
      interval: 3s
      retries: 10
```
{: file='docker-compose.yml'}

Перезапускаем контейнер командой:
```bash
sudo docker compose up -d
```
и проверяем состояние контейнера:
```bash
sudo docker ps
```
Успешный запуск контейнера выдаст приблизительно следующее:
```bash
CONTAINER ID   IMAGE            COMMAND                  CREATED              STATUS                        PORTS                               NAMES
93e8cec33461   traefik:latest   "/entrypoint.sh --ap…"   About a minute ago   Up About a minute (healthy)   0.0.0.0:80->80/tcp, :::80->80/tcp   docker-traefik-1
```
На данном этапе важно значение `STATUS`. Для успешно запущенного и рабочего контейнера оно должно иметь значение `healthy`.
