---
layout: post
title: Curl - работа с HTTP запросами из командной строки.
description: Утилита curl – это мощный инструмент для работы с HTTP-запросами и взаимодействия с различными API
categories:
- Linux
- Консольные приложения
tags:
- linux
- curl
- bash
date: 2024-11-06 11:34 +0300
---
Утилита `curl` – это мощный инструмент для работы с HTTP-запросами и взаимодействия с различными API. В Debian Linux `curl` позволяет скачивать файлы, отправлять данные на сервер, выполнять аутентификацию и работать с API.

## 1. Установка curl на Debian Linux

Для установки `curl` выполните команду:

```bash
sudo apt update
sudo apt install curl
```

После установки можно проверить его версию:

```bash
curl --version
```

## 2. Основное использование curl

### 2.1 Загрузка страницы или файла

Чтобы загрузить веб-страницу или файл, укажите URL:

```bash
curl https://example.com
```

Для сохранения загружаемого файла используйте флаг `-o`:

```bash
curl -o example.html https://example.com
```

Вы также можете использовать `-O` (с большой буквы), чтобы сохранить файл с его исходным именем:

```bash
curl -O https://example.com/file.zip
```

### 2.2 Передача заголовков запроса

Добавить кастомные заголовки (например, `User-Agent`, `Authorization`, `Content-Type`) можно с помощью флага `-H`:

```bash
curl -H "User-Agent: Mozilla/5.0" -H "Accept: application/json" https://example.com
```

## 3. Работа с API: Отправка данных

### 3.1 Отправка данных методом GET

Метод `GET` используется по умолчанию, но вы можете передать параметры запроса прямо в URL:

```bash
curl "https://api.example.com/data?name=John&age=30"
```

### 3.2 Отправка данных методом POST

Для отправки данных методом `POST` используйте `-X POST` и флаг `-d` для передачи данных:

```bash
curl -X POST -d "name=John&age=30" https://api.example.com/data
```

Если API принимает данные в формате JSON, укажите заголовок `Content-Type` и передайте данные в JSON-формате:

```bash
curl -X POST -H "Content-Type: application/json" -d '{"name": "John", "age": 30}' https://api.example.com/data
```

### 3.3 Отправка файла с помощью POST

Для отправки файла используйте флаг `-F`:

```bash
curl -X POST -F "file=@/path/to/file.txt" https://api.example.com/upload
```

## 4. Авторизация

### 4.1 Базовая авторизация

Для базовой авторизации используйте флаг `-u` и укажите логин и пароль:

```bash
curl -u username:password https://api.example.com/protected
```

### 4.2 Bearer Token (JWT)

При работе с токенами (например, JWT) укажите его в заголовке:

```bash
curl -H "Authorization: Bearer your_token_here" https://api.example.com/protected
```

## 5. Дополнительные параметры

### 5.1 Отображение заголовков ответа

Чтобы увидеть заголовки ответа, используйте флаг `-i`:

```bash
curl -i https://example.com
```

### 5.2 Только заголовки ответа

Чтобы получить только заголовки, используйте флаг `-I`:

```bash
curl -I https://example.com
```

### 5.3 Ограничение скорости загрузки

Скорость загрузки можно ограничить, указав значение в `--limit-rate` (например, `200K` или `1M`):

```bash
curl --limit-rate 200K -O https://example.com/file.zip
```

### 5.4 Продолжение загрузки файла

Если загрузка файла была прервана, можно продолжить ее с флагом `-C -`:

```bash
curl -C - -O https://example.com/largefile.zip
```

### 5.5 Игнорирование SSL-сертификатов

Чтобы игнорировать ошибки SSL-сертификата (не рекомендуется для рабочих серверов), используйте флаг `-k`:

```bash
curl -k https://example.com
```

## 6. Примеры использования

### 6.1 Скачивание изображения

```bash
curl -o image.jpg https://example.com/image.jpg
```

### 6.2 Отправка POST-запроса с JSON и Bearer Token

```bash
curl -X POST -H "Authorization: Bearer your_token" -H "Content-Type: application/json" -d '{"key": "value"}' https://api.example.com/endpoint
```

### 6.3 Загрузка файла с прокси-сервером

Укажите прокси-сервер с флагом `-x`:

```bash
curl -x http://proxy.example.com:8080 -O https://example.com/file.zip
```

### 6.4 Использование cookie-файла

Сохранение куки:

```bash
curl -c cookies.txt https://example.com
```

Использование сохраненных куки:

```bash
curl -b cookies.txt https://example.com/protected
```

### 6.5 Отправка нескольких параметров POST

```bash
curl -X POST -d "param1=value1" -d "param2=value2" https://example.com/submit
```

## 7. Подробный вывод для отладки

Для отладки и анализа выполнения `curl` используйте флаг `-v` для детализированного вывода:

```bash
curl -v https://example.com
```

Или `--trace` для еще более подробного вывода:

```bash
curl --trace trace_output.txt https://example.com
```
