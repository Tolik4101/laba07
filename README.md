# Ход работы
Основной задачей данной лабораторной работы было научиться работать с системой автоматизации развёртывания и управления приложениями под названием Docker. Чтобы развернуть свой контейнтер, достаточно создать Dockerfile с заданной в нём конфигурацией систем. Контейнер же, в свою очередь, запускается независимо и изолированно от основной системы, что позволяет побороть проблему платформозависимости и повторяемости окружения.


## Шаг 1. Написание Docker-файла
Для работы Докера требуется конфигурационный файл. Ниже следует сам файл с описанием каждого шага инициализации и запуска контейнера:

```Docker
FROM ubuntu:18.04 #Выбор базового образа операционной системы 
RUN apt update # Обновим информацию о пакетах. пакеты и зависимости
RUN apt install -yy gcc g++ cmake # Установим компиляторы для c\c++ и систему сборки Cmake
COPY . ./print/ # Скопируем локальную директорию в контейнер
WORKDIR ./print # Сделаем её рабочей
# Следующие три шага отвечают за сборку проекта
RUN cmake -H. -B_build  -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=_install # 
RUN cmake --build _build
RUN cmake --build _build --target install
ENV LOG_PATH /home/logs/log.txt # Установим путь для логов (для ошибок и прочей информации)
VOLUME /home/logs   # указываем внешний путь как том
WORKDIR _install/bin # Указываем рабочую директорию
ENTRYPOINT ./demo # Указываем точку входа, она же - исполняемый файл _install/bin/demo
```

## Шаг 2. Настройка Github actions
Для тестирования системы развертывания напишем yml файл с инструкцией по запуску.
Тестирование происходит в два этапа:
### Этап 1: установка Docker на машину github
```sh
$ sudo apt-get install ca-certificates curl gnupg lsb-release
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
$ echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
$ sudo docker run hello-world
$ sudo apt-get install docker
```
### Этап 2: сборка и запуск приложения

```sh
$ docker build -t logger
```
