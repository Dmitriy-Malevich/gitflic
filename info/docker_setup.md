## Запуск в Docker

---

## Linux

**Установка Docker**

Самый простой способ установить Docker на Linux, скачать и выполнить [официальный скрипт](https://docs.docker.com/engine/install/ubuntu/#install-using-the-convenience-script).
* Скачайте и установите curl.
```
sudo apt update
sudo apt install curl
```
* С помощью curl скачайте скрипт для установки докера с официального сайта.
```
curl -fSL https://get.docker.com -o get-docker.sh 
```
* Запустите сохранённый скрипт.
```
sudo sh ./get-docker.sh
```
Если в консоли появятся сообщения об ошибках, установите Docker вручную. Инструкция по установке есть [в официальной документации Docker](https://docs.docker.com/engine/install/ubuntu/).
* Установите утилиту Docker Compose.
```
sudo apt-get install docker-compose-plugin 
```
* Проверьте работу Docker.
```
sudo systemctl status docker
```

**Настройка параметров**

Вам понадобится настроить переменные окружения, находящиеся в `ENV` файле. В качестве теста вы можете использовать настройки указанные по умолчанию.

> **Примечание**: GitFlic запускает собственные ssh сервер на 22 порту для работы сервиса git. Если у Вас данный порт занят вы можете:
> 1. Поменять порт для ssh на вашем сервере. 
>`sudo nano /etc/ssh/sshd_config`
>Раскоментировать параметр `#Port 22` и заменить `22` на другой порт, например `2222`
>Применить изменения командой `sudo systemctl restart sshd.service`
>❗️Если вы подключены к сервере по  `ssh`  то текущая сессия останется рабочей, а при новом подключении не забудьте указать новый порт, например `ssh -p 2222 user@111.222.333.4444`
>2. Поменяйте параметр `HOST_SSH_PORT` в `ENV`. Наприме с `22` на `1234`. 

> **Примечание**: При необходимости более глубокой конфигурации вы можете настроить приложение в конфигурационном `docker/default-config/application.properties` и в `docker_compose.yaml` файлах по инструкции для `application.properties` в [этой статье](setup_and_start).

> **Примечание**: переменная `POSTGRES_CONTAINER_NAME` должна совпадать с названием контейнера `postgres`, указанным в `docker_compose.yaml` файле

**Запуск docker compose**

После настройки перейдите в директорию `docker` и выполните следующую команду:
>Дополнительные ключи. 
>Ключ `--detach`: запускает контейнеры в фоновом режиме.
>Ключ `--build`: запускает все контейнеры, пересобирая их при необходимости. 

```
sudo docker compose --env-file ./ENV up
```

**Стандартный пользователь и пароль:**

* Почта: adminuser@admin.local
* Пароль: qwerty123

## Windows 

* Установите подсистему Linux (WSL2). Информация по установке доступна на [официальном сайте](https://learn.microsoft.com/ru-ru/windows/wsl/install) microsoft.
* Скачайте и установите Docker для windows [https://www.docker.com/get-started/](https://www.docker.com/get-started/)
* После установки запустите приложение **Docker Desktop**
* Запустите оболочку powershell и зайдите в папку **docker** где находится файл `docker-compose.yml` командой
```
cd YOR_PATH\docker
```
* Запустите сборку контейнеров командой 
```
docker compose --env-file ./ENV up
```
После успешного создания  контейнеров и запуска приложения вы увидите сообщение в терминале формата
```
INFO 1 --- [main] c.g.onpremise.OnPremiseApplication       : Started OnPremiseApplication in 23.769 seconds (JVM running for 24.934)
```
Теперь можно зайти на сайт GitFlic по адресу localhost:8080

**Стандартный пользователь и пароль:**

* Почта: adminuser@admin.local
* Пароль: qwerty123

### Загрузка образов Docker из dr.gitflic.ru:
<p>
<details>
<summary>Загрузка образов Docker из dr.gitflic.ru.</summary>

В случае если у Вас нет доступа к https://hub.docker.com/ вы можете скачать необходимые образы докер с нашего сервера dr.gitflic.ru
```
docker pull dr.gitflic.ru/openjdk:11.0.14.1-jdk
docker pull dr.gitflic.ru/postgres:12
docker pull dr.gitflic.ru/redis:6.2-alpine
```
Так же доступны следующие образы
```
docker pull dr.gitflic.ru/redis:7.0.0
docker pull dr.gitflic.ru/runner-helper
```


</details>
</p>
