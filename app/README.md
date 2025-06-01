## Подготовка

* В первую очередь, необходимо установить docker, docker-compose-plugin ([по инструкции](https://docs.docker.com/engine/install/ubuntu/)) и git
* Добавить пользователя в группу docker: `sudo usermod -aG docker $USER` и перелогиниться
* Скачать этот репозиторий `git clone https://github.com/sakharovaan/yandex-kafka-production.git`
* Скачать сертификат yandex cloud `wget "https://storage.yandexcloud.net/cloud-certs/CA.pem" --output-document yandex-kafka-production/app/ca.crt`
* Посмотреть в консоли Yandex Cloud хост для подключения к кластеру Kafka и создать файл `yandex-kafka-production/app/.env` со следующим содержимым (заменив имя хоста и пароль на свои):
```
PRODUCER_KAFKA_BOOTSTRAP_SERVERS=rc1a-96eh53qo92ss79ic.mdb.yandexcloud.net:9091
PRODUCER_SCHEMA_REGISTRY_SERVER=https://rc1a-96eh53qo92ss79ic.mdb.yandexcloud.net:443
PRODUCER_KAFKA_PASSWORD=...

CONSUMER_KAFKA_BOOTSTRAP_SERVERS=rc1a-96eh53qo92ss79ic.mdb.yandexcloud.net:9091
CONSUMER_SCHEMA_REGISTRY_SERVER=https://rc1a-96eh53qo92ss79ic.mdb.yandexcloud.net:443
CONSUMER_KAFKA_PASSWORD=...
```
* Выполнить `cd yandex-kafka-production/app`
* Выполнить `docker compose up -d`

## Тестирование

* Пошлём сообщения через producer: `for i in $(seq 1 100); do curl -vv -XPOST -d '{"text": "message"}' -H "Content-Type: application/json" localhost:9000/api/v1/kafka/send; done`