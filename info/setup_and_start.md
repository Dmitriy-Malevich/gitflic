# Установка и запуск GitFlic 

---
[Скачайте](https://gitflic.ru/project/gitflic/gitflic/release/latest) последнюю версию GitFlic self-hosted.

Для того чтобы запустить приложение вам понадобится установить следующие приложения:
 * Java 11 (протестировано на openjdk11)
 * Redis (протестировано на версии 6.2)
 * PostgreSQL (протестировано на версии 11 и 12). 
 * Elasticsearch 7.16.2 (опционально)
 * RebbitMQ 3.12  (для enterprice версии)

Для корректной работы приложения необходимо при конфигурации базы данных PostgreSQL установить расширение pgcrypto, для этого необходимо **для конкретной базы данных** в СУБД выполнить следующий запрос:

`CREATE EXTENSION pgcrypto;`

Так же необходимо сконфигурировать SMTP сервер, для отправки писем из сервиса и сгенерировать ключ `key.pem` для работы ssh транспорта git.

Схема работы приложения GitFlic  будет выглядеть следующим образом:

<img src="https://gitflic.ru/project/gitflic/docs/blob/raw?file=img%2Fsetup%2Fschema-2.jpg&commit=99b34a89f45e5f950642e51b881cbec8e8cc6f63" alt="Схема">

---

## Установка Java 11

* Ubuntu и debian-based дистрибутивы: 
```
sudo apt-get install openjdk-11-jdk
```
* [РЕД ОС](https://redos.red-soft.ru/base/arm/arm-other/java-openjdk/) и RedHat дистрибутивы: 
```
sudo dnf install java-11-openjdk -y
```
* [Astra Linux](https://wiki.astralinux.ru/pages/viewpage.action?pageId=147162398) (в некоторых случаях придётся скачивать и устанавливать пакет вручную)
* [Альт Сервер](https://packages.altlinux.org/ru/sisyphus/srpms/java-11-openjdk/) все команды в alt-server выполняются под root `su -`
* Для запуска gitflic на **Windows** рекомендуем воспользоваться запуском через [docker compose](https://docs.gitflic.space/setup/docker_setup#windows). Файлы для запуска в докере находятся в архиве рядом с дистрибутивом.
```
apt-get update && apt-get install java-11-openjdk
```
Если возникнут проблемы с установкой java 11 по умолчанию воспользутесь следующими рекоммендациями.
<p>
<details>
<summary>Если возникнут проблемы с установкой java 11 по умолчанию</summary>

Проверьте какая версия java доступна у Вас по умолчанию командой:
```
java --version
```
Если версия java не 11, то Вы можете удалить не нужную версию, если она Вам не нужна.
Посмотрите какие версии java у Вас установлены командой:
```
rpm -qa --qf '%{name}\n' | grep java
```
Удалите не нужную версию командой:
```
apt-get remove пакет
```

</details>
</p>

<p>
<details>
<summary>Проверить версию java</summary>

Проверить версию java можно с помощью команды:
```
java --version
```

</details>
</p>

---

## Установка postgresql

* [Ubuntu и debian-based](https://www.postgresql.org/download/linux/ubuntu/) дистрибутивы: 
```
sudo apt-get install postgresql-12 postgresql-contrib
```

<p>
<details>
<summary>Добавление дополнительных репозиториев для Ubuntu</summary>

```
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'  
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -  
sudo apt-get update  
sudo apt-get install postgresql-12 postgresql-contrib
```

</details>
</p>

* [РЕД ОС](https://redos.red-soft.ru/base/server-configuring/dbms/install-postgresql/#postgresql12) и [RedHat](https://www.postgresql.org/download/linux/redhat/) дистрибутивы:
```
sudo yum install postgresql-server postgresql-contrib
```
* [Аstra Linux](https://wiki.astralinux.ru/pages/viewpage.action?pageId=147162402) (в некоторых случаях придётся скачивать и устанавливать пакет вручную)
* [Альт Сервер](https://www.altlinux.org/PostgreSQL) все команды в alt-server выполняются под root `su -`
```
apt-get install postgresql12-server postgresql12-contrib
```

## Запуск postgresql

* Инициализация сервера (создание системных баз данных):
* Ubuntu и debian-based дистрибутивы: 
```
sudo /etc/init.d/postgresql initdb
```
* РЕД ОС
```
sudo postgresql-setup initdb
```
* [Альт Сервер](https://www.altlinux.org/PostgreSQL)  все команды в alt-server выполняются под root `su -`. Запуск сервера происходит как в других версиях только без использования `sudo`
```
/etc/init.d/postgresql initdb
```

* Запуск сервера:
```
sudo systemctl enable postgresql
sudo systemctl start postgresql
sudo systemctl status postgresql
```

### Конфигурация postgresql
* Ubuntu и debian-based, РЕД ОС
* Зайдите под пользователем PostgreSQL:
```
sudo -i -u postgres
# для выхода
\q
```
* Создайте пользователя и базу данных:
```
createuser gitflic
createdb gitflic
```
* Войдите в базу данных:
```
psql --dbname "gitflic"
```
* Дайте пароль пользователю:
```
alter user gitflic with password 'gitflic';
```
* Загрузите расширение pgcrypto:
```
CREATE EXTENSION pgcrypto;
```
* [Альт Сервер](https://www.altlinux.org/PostgreSQL)
* Создайте пользователя и базу данных:
```
createuser -U postgres --no-superuser --no-createdb --no-createrole --encrypted --pwprompt gitflic

createdb -U postgres -O gitflic gitflic
```
* Загрузка расширения pgcrypto:
```
psql -U postgres -d gitflic
# Выход по Ctrl+D или командой quit

alter user gitflic with password 'gitflic';

CREATE EXTENSION pgcrypto;
```
<p>
<details>
<summary>Возможные ошибки</summary>

Если при загрузке расширения выходит ошибка, то надо проверить, что у Вас установилась postgresql-contrib

</details>
</p>

* В файле /var/lib/pgsql/data/pg_hba.conf  или /etc/postgresql/12/main/pg_hba.conf
замените строку
`host all all 127.0.0.1/32 ident`
на
`host all all 127.0.0.1/32 md5`
для использования аутентификации по паролю.

<p>
<details>
<summary>Возможные ошибки</summary>

Если не получается найти файл pg_hba.conf, то выполняем следующие команды:
```
sudo -i -u postgres
psql -t -P format=unaligned -c 'show hba_file';
```

</details>
</p>

* Перезагрузите сервер:
```
service postgresql restart
```

---

## Установка Redis

* [Ubuntu и debian-based](https://redis.io/docs/getting-started/installation/install-redis-on-linux/) дистрибутивы: 
```
sudo apt-get install redis
```
* РЕД ОС и RedHat дистрибутивы: 
```
sudo dnf install redis -y
```
* Альт Сервер
```
apt-get install redis
```

Запуск Redis:

```
# Команда для запуска сервиса Redis
systemctl enable redis
# Для добавления Redis в автозагрузку, введите команду
systemctl start redis

```

<p>
<details>
<summary>Проверка работы Redis</summary>

Проверить версию Redis 
```
redis-server --version
```
Проверить состояние Redis
```
sudo systemctl status redis
```
Убедиться, что Redis запущен, можно с помощью команды (должна вернуть PONG)
```
redis-cli ping
```

</details>
</p>

---

## Настройка SSH порта

Для того, чтобы было возможным использовать remote-url вида `git@gitflic.ru:gitflic/gitflic.git` ssh сервер должен быть запущен на стандартном для ssh соединения порту 22.

Во избежание конфликта с системным ssh, который слушает 22 порт, настройте iptables для перенаправления 22 порта на любой другой порт для работы ssh gitflic и укажите данный порт в `application.properties`.

```
sudo iptables -t nat -I PREROUTING -p tcp --dport 22 -j REDIRECT --to-ports 1122
sudo iptables -t nat -I OUTPUT -p tcp -o lo --dport 22 -j REDIRECT --to-ports 1122
```

все входящие пакеты на порт 22 будут перенаправлены на порт 1122, так что при обращении к серверу необходимо указывать порт.

<p>
<details>
<summary>Изменение настроек ssh</summary>

В качестве альтернативы можно изменить порт для подключения по ssh

[РЕД ОС](https://redos.red-soft.ru/base/manual/admin-manual/utilites/ssh/)
```
# меняем порт например на 2222
sudo nano /etc/ssh/sshd_config 
# применяем изменения в системе безопасности. Если их не применить система заблокирует изменения порта.
sudo semanage port -a -t ssh_port_t -p tcp 2222
sudo systemctl restart sshd
```
Возможно проблема с [semanage](https://itsecforu.ru/2020/02/10/%F0%9F%90%A7-%D1%83%D1%81%D1%82%D1%80%D0%B0%D0%BD%D0%B5%D0%BD%D0%B8%D0%B5-%D0%BD%D0%B5%D0%BF%D0%BE%D0%BB%D0%B0%D0%B4%D0%BE%D0%BA-%D0%B2-linux-semanage-command-not-found-%D0%BD%D0%B0-centos-7-8/)
Для устрания проблемы с semanage в РЕДОС 7.3 были выполнены следующие команды
```
dnf install policycoreutils-python-utils
```
Альт сервер
```
nano /etc/openssh/sshd_config
# Раскоментировать строку Port и задать порт отличный от 22, например 2222
systemctl restart sshd
```

</details>
</p>

---

## Генерация key.pem

Для генерации ssh ключа откройте терминал и выполните команду:

```
ssh-keygen -t ed25519
```

В консоль будет выведен следующий диалог:
r
`Enter file in which to save the key (/home/user/.ssh/id_ed25519):`

Нажмите на клавишу Enter. 

Cистема предложит ввести кодовую фразу для дополнительной защиты SSH-подключения. (Данный шаг можно пропустить.)

`Enter passphrase (empty for no passphrase):`

После этого ключ будет создан и помещён в директорию `/home/user/.ssh/`. Перейдите в директорию `/home/user/.ssh` и скопируйте файл с приватным ключом `id_ed25519` в директорию `cert/`, после переименуйте его в `key.pem`.
Папку `cert/` можно создать где Вам удобно, путь к сертификату Вы укажите при конфигурации файла `application.properties`.
Или можно разместить ключ в директорию которая указана по умолчанию в application.properties. Для этого можно выполнить следующие команды из папки .ssh. Кроме ключа будут созданы все папки, которые указаны в `application.properties` по умолчанию.
```
mkdir -p /opt/gitflic/var/cert;
cp id_ed25519 /opt/gitflic/var/cert/key.pem;
cd /opt/gitflic/var;
mkdir repo img releases cicd registry
```

Так же вы можете воспользоваться командами для созания ключа без дополнительных параметров.
<p>
<details>
<summary>Создание приватного ключа и копирование его в папку tmp/gitflic/cert</summary>

```
ssh-keygen -t ed25519 -f id_ed25519 -N ""
mkdir -p /opt/gitflic/var/cert
cp ~/.ssh/id_ed25519 /opt/gitflic/var/cert/key.pem
cd /opt/gitflic/var/
mkdir repo img releases cicd registry
```

</details>
</p>

---

## Установка RabbitMQ
☘️ Данное ПО устанавливается для платной версии [self-hosted enterprise](https://gitflic.ru/price).
RabbitMQ - это распределённый и горизонтально масштабируемый брокер сообщений, является одним из самых популярных средств для обмена данными между сервисами. Благодаря использованию RabbitMQ увеличивается отказоустойчивость и количество обрабатываемых сообщений в GitFlic.

* Ubuntu и debian-based дистрибутивы: 
```
sudo apt-get update -y
```
Установка необходимых пакетов erlang
```
sudo apt-get install -y erlang-base \
                        erlang-asn1 erlang-crypto erlang-eldap erlang-ftp erlang-inets \
                        erlang-mnesia erlang-os-mon erlang-parsetools erlang-public-key \
                        erlang-runtime-tools erlang-snmp erlang-ssl \
                        erlang-syntax-tools erlang-tftp erlang-tools erlang-xmerl
```
Проверить успешную установку erlang.
<p>
<details>
<summary>Проверить успешную установку erlang.</summary>

выполните команду 
```
erl
```
для выхода из консоли нажмите **Ctrl+C**

</details>
</p>
Установка RabbitMQ
```
sudo apt-get install rabbitmq-server -y --fix-missing
```
После установки `RabbitMQ` сам запускается и добавляется в автозагрузку. Ниже приведены команды которые позволят это сделать вручную
```
sudo systemctl status RabbitMQ-server.service 
sudo systemctl start rabbitmq-server.service 
sudo systemctl enable rabbitmq-server.service 
```

* Для РЕД ОС и RedHat дистрибутивов инструкцию можно посмотреть [здесь](https://www.rabbitmq.com/install-rpm.html) 
* Альт Сервер. [Здесь](https://www.altlinux.org/%D0%A0%D1%83%D0%BA%D0%BE%D0%B2%D0%BE%D0%B4%D1%81%D1%82%D0%B2%D0%BE_%D0%BF%D0%BE_%D1%83%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B5_%D0%B8_%D0%B7%D0%B0%D0%BF%D1%83%D1%81%D0%BA%D1%83_OpenStack_%D0%B2_ALT_Linux_%D0%B8_%D0%BF%D1%80%D0%BE%D0%B8%D0%B7%D0%B2%D0%BE%D0%B4%D0%BD%D1%8B%D1%85) есть рекомендация по установке RabbitMQ.

---

## Конфигурация application.properties

В данном разделе находится информация по поводу конфигурации `default-config/application.properties` файла.

Файл конфигурации находится в архиве вместе с приложением. Скачать приложение вы можете по [ссылке](https://gitflic.ru/project/gitflic/gitflic/release/latest).

> Примечание: Дополнительные параметры являются необязательными.

### Конфигурация сервера

* `server.port` отвечает за порт, на котором будет запущено приложение Gitflic.
    - Стандартное значение: `8080`
* `server.address` отвечает за IP-адрес, на котором будет доступно приложение Gitflic для входа в браузере. (по умолчанию localhost)
    - Стандартное значение: `localhost`
* `ssh.server.port` отвечает за порт, на котором будет запущен ssh сервер для обращения к Gitflic по remote-url вида `git@gitflic.ru:gitflic/gitflic.git` 
    - Стандартное значение: `22`

**Дополнительные параметры:**

* `spring.servlet.multipart.maxFileSize` отвечает за размер загружаемых файлов в формах и запросах api
* `spring.servlet.multipart.maxRequestSize` отвечает за размер запроса 
    - Стандартное значение для обоих параметров: `500MB`

### Конфигурация базы данных

* `spring.datasource.url` отвечает за url для подключения к базе данных.
    - Стандартное значение: `jdbc:postgresql://localhost/gitflic`
* `spring.datasource.username` отвечает за имя пользователя для аутентификации в базе данных.
    - Стандартное значение: `gitflic`
* `spring.datasource.password` отвечает за пароль пользователя для аутентификации в базе данных.
    - Стандартное значение: `gitflic`

### Конфигурация Redis

* `spring.redis.host` отвечает за адрес, на котором запущен Redis.
    - Стандартное значение: `localhost`
* `spring.redis.port` отвечает за порт, на котором запущен Redis.
    - Стандартное значение: `6379`

**Дополнительные параметры:**

* `spring.redis.username` отвечает за имя пользователя для аутентификации в Redis.
* `spring.redis.password` отвечает за пароль для аутентификации в Redis.
* `spring.redis.database` отвечает за номер базы данных.
    - Стандартное значение: `0`
* `spring.redis.ssl`. Если редис находится не в локальном контуре, а обращаться к нему необходимо через https протокол,
то возможно указать данный параметр
    - Стандартное значение: `false`
* Так же возможно указать полный URL для соединения с Redis.
    -  Если указан URL для соединения, то параметры `spring.redis.host`, `spring.redis.port`, и `spring.redis.password` будут перезаписаны из URL. Параметр `user` будет проигнорирован. Пример URL: `redis://user:password@example.com:6379`


### Настройки директорий, используемых для работы приложения

Пути к директориям должны заканчиваться на `/`

* `repository.dir` отвечает за директорию, где должны храниться репозитории git.
 - Стандартное значение: `/opt/gitflic/var/repo/`
* `image.upload.dir` отвечает за директорию, где должны храниться аватары и иные медиафайлы.
 - Стандартное значение: `/opt/gitflic/var/img/`
* `releases.upload.dir` отвечает за директорию, где должны храниться файлы, которые приложены к релизу на основе тегов git.
 - Стандартное значение: `/opt/gitflic/var/releases/`
* `ssh.server.cert` отвечает за путь к сертификату, который используется для ssh транспорта.
 - Стандартное значение: `/opt/gitflic/var/cert/key.pem`
* `cicd.pipeline.dir` отвечает за директорию, которая используется CI/CD агентом.
    - Стандартное значение: `/opt/gitflic/var/cicd/`
- `gitflic.registry.package.dir` отвечает за путь для файлов реестра пакетов.
    * Стандартное значение: `/opt/gitflic/var/registry/`
Вы можете создать указанные выше директории командой 

```
mkdir -p /opt/gitflic/var/cert;
cd /opt/gitflic/var;
mkdir repo img releases cicd registry
```

> После запуска убедитесь, что все обозначенные директории созданы в вашем хранилище.

### Настройка SMTP сервера

❗️Без настройки smtp сервера приложение не сможет отправлять письма как минимум для регистрации пользователей, однако в данном случае возможно создать пользователей через админ панель в приложении.

Для использования SMTP используйте любое подходящее под вашу операционную систему приложение.

[Как настроить локальный SMTP-сервер](https://habr.com/ru/post/544376/)

* `spring.mail.host` отвечает за адрес SMTP сервера.
* `spring.mail.port` отвечает за порт SMTP сервера.
* `spring.mail.username` отвечает за имя пользователя для аутентификации на SMTP сервере.
* `spring.mail.password` отвечает за пароль для аутентификации на SMTP сервере.
**Дополнительные параметры:**
Данные параметры могут Вам пригодиться если вы планируете использовать общедоступные сервисы для отправки сообщений, например mail.yandex.ru. Обратите внимание, что при такой настройке вам необходимо получить специальный пароль для приложений, а не использовать стандартный пароль ([документация яндекс](https://yandex.ru/support/id/authorization/app-passwords.html)).
* `smtp.sender.name=testname`
* `smtp.sender.email=testname@gitflic.ru`
Параметры `starttls` и `ssl` взаимоисключающие.
* `spring.mail.properties.mail.smtp.starttls.enable = true`
* `spring.mail.properties.mail.smtp.ssl.enable=true`
* `spring.mail.properties.mail.smtp.auth=true`

### Общие свойства приложения

* `gitflic.base.url` отвечает за генерацию ссылок для внешних источников, например для ссылок в письмах (Стандартное значение для данного свойства localhost).
    - Данное свойство должно содержать в себе домен или хост, при необходимости, порт, на котором запущено приложение. Например, данное свойство для SaaS варианта нашего сервиса имеет значение `gitflic.base.url=https://gitflic.ru`. При отсутствии протокола передачи данных по умолчанию будет использоваться `https://`. 
* `gitflic.transport.url` отвечает за генерацию урла, который отображается на странице каждого проекта и приходит в данных из методов API и в данных вебхуков (Стандартное значение для данного параметра localhost).
    - Данное свойство должно содержать в себе домен или хост, по которому осуществляется транспорт данных в git репозиторий. 
    Есть некоторые нюансы в генерации урлов для ssh транспорта, так как порт, по которому осуществляется биндинг ssh сервера - 22, но может быть изменен, то вариант урла, например, `git@gitflic.ru:vault/zookeeper.git` с нестандартными настройками работать не будет. 
 В таком случае необходимо настроить прокси-сервер (например HaProxy), для того, чтобы внешние обращения были перенаправлены на указанный в приложении порт.
 Плюс если указать в значении урла порт (например localhost:8080), то для ssh транспорта будет сгенерирован
 урл `git@localhost:vault/zookeeper.git` без указанного порта.

### Дополнительные параметры:

* `gitflic.defaultPackSize` ограничивает максимальный размер пакета, который гит может отправить во время пуша в репозиторий.
    - Стандартное значение 100MB. Данное поле имеет тип String. Возможны следующие суффиксы KB, MB, GB, TB.
* `gitflic.limitPackSize` используется для включения и отключения механизма ограничения максимального размера
пакета при пуше. Если данный параметр имеет значение true, то настройка конкретного проекта происходит через настройки
компании в администраторской панели.
* `gitflic.limitProjectSize` использется для включения и отключения механизма ограничения максимального размера репозитория. Если данное значение имеет значение true, то настройка конретного проекта происходит через настройки проекта в панели администрирования.
* `logging.file.name` отвечает за название файла логов 
* `logging.file.path` отвечает за директорию для файла логов
* `logging.level.root` отвечает за настройку уровня логирования
    - `OFF` - логирование выключено
    - `ERROR` - показывает ошибки значительной важности, которые помешают нормальному выполнению программы
    - `WARN` - показывает предупреждения 
    - `INFO` - показывает информационные сообщения, которые могут иметь смысл для конечных пользователей и системных администраторов, освещающие ход работы приложения
    - `DEBUG` - показывает детальное отслеживание, используемое разработчиками приложений. 
    - `ALL` - вывод всех сообщений

---

## Запуск через JAR-файл

Для запуска приложения в консоли необходимо выполнить следующую команду из директории, где находится исполняемый файл `gitflic.jar`:

```
java -jar gitflic.jar --spring.config.additional-location=file:default-config/
```

Обратите внимание, что в примере указана директория с конфигурационным файлом относительно директории, в которой расположен
jar пакет. Вы можете переместить конфиг файл в любую удобную вам директорию и указать к ней путь в параметре
`--spring.config.additional-location`. Обратите внимание, что путь к директории должен оканчиваться символом /

Стандартный пользователь и пароль:

* Почта: adminuser@admin.local 
* Пароль: qwerty123

При возникновении проблем с запуском приложения, рекомендуем вам ознакомиться со [страницей](https://docs.gitflic.space/faq/faq_gitflic), где собраны ошибки, с которыми сталкивались пользователи. 

---

## Балансировка нагрузки (Кластеризация)

Есть возможность настроить приложение GitFlic с несколькими инстансами.
Схема работы приложения GitFlic с балансировкой будет выглядеть следующим образом:

<img src="https://gitflic.ru/project/gitflic/docs/blob/raw?file=img%2Fsetup%2Fschema-3.jpg&branch=master" alt="Схема с балансировкой">

---

## Реестр пакетов

В нашем приложении из-за уязвимости связанной с использованием косой черты в кодировки URL (%2F) ([источник](https://github.com/advisories/GHSA-4prh-gqw8-rgh5)) включен файрвол который который блокирует потенциально опасные запросы. В связи с этим, не будут работать реестры пакетов npm, пока не будет настроен nginx в качестве обратного прокси сервера.

✅ Рекомендуемый вариант?
Для работы с реестром пакетов npm вам нужно настроить nginx в качестве обратного прокси сервера для приложения GitFlic.

❌ Не рекомендуемый вариант:
В качестве альтернативы можно отключить внутренний файрвол. Рекомендуем не использовать данный вариант.
* Внесите дополнительную настройку в `application.properties`
```
togglz.features.GITFLIC_FIREWALL_SLASH.enabled=true 
```
* Запустить приложение с дополнительным ключом 
```
-Dorg.apache.tomcat.util.buf.UDecoder.ALLOW_ENCODED_SLASH=true
```
