# Домашнее задание к занятию 5. «Практическое применение Docker»

## Задача 0
1. Убедитесь что у вас НЕ(!) установлен ```docker-compose```, для этого получите следующую ошибку от команды ```docker-compose --version```
```
Command 'docker-compose' not found, but can be installed with:

sudo snap install docker          # version 24.0.5, or
sudo apt  install docker-compose  # version 1.25.0-1

See 'snap info docker' for additional versions.
```
![0_1](https://github.com/AlekseyStroitelev/docker-practica/blob/main/screenshots/docker0_1.png)

В случае наличия установленного в системе ```docker-compose``` - удалите его.  
2. Убедитесь что у вас УСТАНОВЛЕН ```docker compose```(без тире) версии не менее v2.24.X, для это выполните команду ```docker compose version```  

![0_2](https://github.com/AlekseyStroitelev/docker-practica/blob/main/screenshots/docker0_2.png)
---

## Задача 1
1. Сделайте в своем github пространстве fork [репозитория](https://github.com/netology-code/shvirtd-example-python/blob/main/README.md).
   Примечание: В связи с доработкой кода python приложения. Если вы уверены что задание выполнено вами верно, а код python приложения работает с ошибкой то используйте вместо main.py файл not_tested_main.py(просто измените CMD)
3. Создайте файл с именем ```Dockerfile.python``` для сборки данного проекта(для 3 задания изучите https://docs.docker.com/compose/compose-file/build/ ). Используйте базовый образ ```python:3.9-slim```. 
Обязательно используйте конструкцию ```COPY . .``` в Dockerfile. Не забудьте исключить ненужные в имадже файлы с помощью dockerignore. Протестируйте корректность сборки.  

### Ответ:

Сделал форк репа и собрал образ.

---

## Задача 2 (*)
1. Создайте в yandex cloud container registry с именем "test" с помощью "yc tool" . [Инструкция](https://cloud.yandex.ru/ru/docs/container-registry/quickstart/?from=int-console-help)
2. Настройте аутентификацию вашего локального docker в yandex container registry.
3. Соберите и залейте в него образ с python приложением из задания №1.
4. Просканируйте образ на уязвимости.
5. В качестве ответа приложите отчет сканирования.

### Ответ:

[vulnerabilities](https://github.com/AlekseyStroitelev/docker-practica/tree/main/scan_result/vulnerabilities.csv)

## Задача 3
1. Изучите файл "proxy.yaml"
2. Создайте в репозитории с проектом файл ```compose.yaml```. С помощью директивы "include" подключите к нему файл "proxy.yaml".
3. Опишите в файле ```compose.yaml``` следующие сервисы: 

- ```web```. Образ приложения должен ИЛИ собираться при запуске compose из файла ```Dockerfile.python``` ИЛИ скачиваться из yandex cloud container registry(из задание №2 со *). Контейнер должен работать в bridge-сети с названием ```backend``` и иметь фиксированный ipv4-адрес ```172.20.0.5```. Сервис должен всегда перезапускаться в случае ошибок.
Передайте необходимые ENV-переменные для подключения к Mysql базе данных по сетевому имени сервиса ```web``` 

- ```db```. image=mysql:8. Контейнер должен работать в bridge-сети с названием ```backend``` и иметь фиксированный ipv4-адрес ```172.20.0.10```. Явно перезапуск сервиса в случае ошибок. Передайте необходимые ENV-переменные для создания: пароля root пользователя, создания базы данных, пользователя и пароля для web-приложения.Обязательно используйте уже существующий .env file для назначения секретных ENV-переменных!

2. Запустите проект локально с помощью docker compose , добейтесь его стабильной работы: команда ```curl -L http://127.0.0.1:8090``` должна возвращать в качестве ответа время и локальный IP-адрес. Если сервисы не стартуют воспользуйтесь командами: ```docker ps -a ``` и ```docker logs <container_name>``` . Если вместо IP-адреса вы получаете ```NULL``` --убедитесь, что вы шлете запрос на порт ```8090```, а не 5000.

![3_2_1](https://github.com/AlekseyStroitelev/docker-practica/blob/main/screenshots/docker3_2_1.png)

![3_2_2](https://github.com/AlekseyStroitelev/docker-practica/blob/main/screenshots/docker3_2_2.png)

5. Подключитесь к БД mysql с помощью команды ```docker exec -ti <имя_контейнера> mysql -uroot -p<пароль root-пользователя>```(обратите внимание что между ключем -u и логином root нет пробела. это важно!!! тоже самое с паролем) . Введите последовательно команды (не забываем в конце символ ; ): ```show databases; use <имя вашей базы данных(по-умолчанию example)>; show tables; SELECT * from requests LIMIT 10;```.

6. Остановите проект. В качестве ответа приложите скриншот sql-запроса.

### Ответ:

![3_4](https://github.com/AlekseyStroitelev/docker-practica/blob/main/screenshots/docker3_4.png)

## Задача 4
1. Запустите в Yandex Cloud ВМ (вам хватит 2 Гб Ram).
2. Подключитесь к Вм по ssh и установите docker.
3. Напишите bash-скрипт, который скачает ваш fork-репозиторий в каталог /opt и запустит проект целиком.
4. Зайдите на сайт проверки http подключений, например(или аналогичный): ```https://check-host.net/check-http``` и запустите проверку вашего сервиса ```http://<внешний_IP-адрес_вашей_ВМ>:8090```. Таким образом трафик будет направлен в ingress-proxy. ПРИМЕЧАНИЕ:  приложение main.py( в отличие от not_tested_main.py) весьма вероятно упадет под нагрузкой, но успеет обработать часть запросов - этого достаточно. Обновленная версия (main.py) не прошла достаточного тестирования временем, но должна справиться с нагрузкой.
5. (Необязательная часть) Дополнительно настройте remote ssh context к вашему серверу. Отобразите список контекстов и результат удаленного выполнения ```docker ps -a```
6. В качестве ответа повторите  sql-запрос и приложите скриншот с данного сервера, bash-скрипт и ссылку на fork-репозиторий.

### Ответ:

Что бы не палить значения переменных в git'e, предварительно закинул файл .env руками.

Скрипт дилетантский, но худо-бедно отработал:

```
#!/bin/bash
REPO="https://github.com/AlekseyStroitelev/docker-practica.git"
DIR="/opt/repo"
apt-get update
apt-get upgrade -y
apt-get install docker.io -y
apt-get install docker-compose-v2 -y
apt-get install git -y
mkdir /opt/repo
git clone "$REPO" "$DIR"
cd "$DIR"
docker compose up -d
```

![4](https://github.com/AlekseyStroitelev/docker-practica/blob/main/screenshots/docker4.png)

https://github.com/AlekseyStroitelev/docker-practica/blob/main/README.md

## Задача 6
Скачайте docker образ ```hashicorp/terraform:latest``` и скопируйте бинарный файл ```/bin/terraform``` на свою локальную машину, используя dive и docker save.
Предоставьте скриншоты  действий.

### Ответ:

С помощью `dive` находим и запоминаем нужный слой в котором фигурирует наш /bin/terraform.
Далее с помощью `docker save` сохраняем образ в виде `*.tar` - файла, распаковываем, в полученном результате ищем директорию /blobs/sha256 и вней находим нужный слой. Еще раз с помощью утилиты `tar` распаковываем теперь уже наш конкретный слой и в результате видим `/bin/terraform`

![6_1](https://github.com/AlekseyStroitelev/docker-practica/blob/main/screenshots/docker6_1.png)

![6_2](https://github.com/AlekseyStroitelev/docker-practica/blob/main/screenshots/docker6_2.png)

## Задача 6.1
Добейтесь аналогичного результата, используя docker cp.  
Предоставьте скриншоты  действий.

### Ответ:

Запускаем контейнер, смотрим его ID и с помощью `docker cp` копируем нужный файл в нужное место.

![6.1](https://github.com/AlekseyStroitelev/docker-practica/blob/main/screenshots/docker6.1.png)

---