## Подготовка

* Скачать сертификат yandex cloud `wget "https://storage.yandexcloud.net/cloud-certs/CA.pem" --output-document yandex-kafka-production/hadoop-consumer/ca.crt`
* Посмотреть в консоли Yandex Cloud хост для подключения к кластеру Kafka и создать файл `yandex-kafka-production/hadoop-consumer/.env` со следующим содержимым (заменив имя хоста и пароль на свои):
```
CONSUMER_KAFKA_BOOTSTRAP_SERVERS=rc1a-96eh53qo92ss79ic.mdb.yandexcloud.net:9091
CONSUMER_SCHEMA_REGISTRY_SERVER=https://kafka-user:...@rc1a-96eh53qo92ss79ic.mdb.yandexcloud.net:443
CONSUMER_KAFKA_PASSWORD=...
```
* Выполнить `cd yandex-kafka-production/hadoop-consumer`
* Выполнить `docker compose up -d`
