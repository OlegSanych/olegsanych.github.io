---
layout: post
title: Установка Docker и Docker Compose на Debian 12
description: Подробная инструкция для установки Docker и Docker Compose на Debian Linux.
image: assets/img/posts/ustanovka-docker-i-docker-compose-na-debian-12/docker.jpg
categories:
- Linux
- Docker
- Debian
- Ubuntu
tags:
- linux
- docker
- debian
- ubuntu
date: 2024-07-19 17:14 +0300
---
## Предварительные условия для установки Docker

Прежде чем начать, необходимо включить аппаратную виртуализацию. Это относится к VT-x на Intel и AMD-V на материнских платах AMD. Это необходимо для запуска Docker.  
На материнских платах AMD AMD-V включена по умолчанию. Однако на материнских платах Intel вам придется вручную включить VT-x в BIOS / UEFI.

## 1. Обновление и установка зависимостей Docker

Сначала обновим список пакетов и установим необходимые зависимости docker.

```bash
sudo apt update
```

## 2. Установка Docker с помощью скрипта.
В официальной [документации Docker](https://docs.docker.com) представлен скрипт для полнотью автоматической установки. Такой способ **не рекомендуктся** для производственных сред, но может быть удобен в тестовых.

> Всегда проверяйте скрипты, загруженные из Интернета, прежде чем запускать их.
{: .prompt-tip }

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
```
```bash
sudo sh ./get-docker.sh --dry-run
```

Эти команды загружает сценарий с сайта https://get.docker.com/ и запускает его для установки последнего стабильного выпуска Docker.

## 3. Установка Docker из репозитория.

Перед тем как устанвливать Docker, необходимо настроить репозиторий. После этого станет возможным устанавливать и обновлять Docker из репозитория.

Устанавливаем необходимые пакеты и зависимости:

```bash
sudo apt install ca-certificates curl
```

## 4. Добавление репозиторя Docker в источники APT

В начале необходимо получить ключ GPG, который требуется для подключения к репозиторию Docker. Для этого выполняем

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```
```bash
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
```
```bash
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Затем добавляем хранилище в список источников.

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Обновляем пакеты.

```bash
sudo apt update
```

## 5. Установка Docker

В этом руководстве устанавливается пакет docker-ce (а не пакет docker.io).

>**В чем разница между docker.io и docker-ce?**  
**docker.io** - это пакет docker, который предлагается некоторыми популярными дистрибутивами Linux (например, Ubuntu/Debian). **docker-ce**, это пакет docker из официального репозитория Docker. Как правило, docker-ce более современен и предпочтителен.

Выполняем следующую команду:

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## 6. Как убедиться, что Docker и Docker Compose установлены успешно?

Существует множество способов это проверить. Один из способов - вывести версию, установленного Docker.

```bash
sudo docker version
```

Должно появиться сообщение с информацией о версии компонентов.
```bash
Client: Docker Engine - Community
 Version:           27.0.3
 API version:       1.46
 Go version:        go1.21.11
 Git commit:        7d4bcd8
 Built:             Sat Jun 29 00:02:50 2024
 OS/Arch:           linux/amd64
 Context:           default
 ```

```bash
sudo docker compose version
```

Должно появиться сообщение с информацией о версии.
```bash
Docker Compose version v2.28.1
```