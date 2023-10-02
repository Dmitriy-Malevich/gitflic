# Процесс обновления GitFlic

---
[Скачайте](https://gitflic.ru/project/gitflic/gitflic/release/latest) последнюю версию GitFlic self-hosted.

Ниже описано, какие дополнительные настройки на сервере надо произвести при переходе на новую версию. Все настройки расписаны под версией продукта. Если для новой версии gitflic нет дополнительной информации, то для обновления достаточно заменить старый gitflic.jar на новый.

### v 2.14.0
#### Общее обновление для всех версий
Добавился новый функционал "Реестр пакетов". Для его работы необходимо добавить новый параметр в `application.properties` и создать папку для хранения пакетов. 
```
mkdir -p /opt/gitflic/var/registry
echo "#Параметр gitflic.registry.package.dir отвечает за путь для файлов реестра пакетов" >> /opt/gitflic/default-config/application.properties
echo "gitflic.registry.package.dir=/opt/gitflic/var/registry/" >> /opt/gitflic/default-config/application.properties
```
#### Дополнительное обновление для платной версии [self-hosted enterprise](https://gitflic.ru/price)
Для платной версии так же необходимо установить сервис RabbitMQ. Инструкцию по установке можно посмотреть [здесь](https://docs.gitflic.space/setup/setup_and_start#%D0%A3%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0-rabbitmq).
Для дополнительных настроек RabbitMQ необходимо добавить дополнительные параметры в `application.properties`
```
# ======= Конфигурация RabbitMQ =======

#spring.rabbitmq.host=localhost
#spring.rabbitmq.port=5672
#spring.rabbitmq.username=
#spring.rabbitmq.password=

#Дополнительные настройки для соединения с RabbitMq можно посмотреть по ссылке
# https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#application-properties.integration.spring.rabbitmq.addresses
```

