## Подготовка

* В первую очередь, необходимо установить docker, docker-compose-plugin ([по инструкции](https://docs.docker.com/engine/install/ubuntu/)) и git
* Добавить пользователя в группу docker: `sudo usermod -aG docker $USER` и перелогиниться
* Скачать этот репозиторий `git clone https://github.com/sakharovaan/yandex-kafka-production.git`
* Скачать сертификат yandex cloud `wget "https://storage.yandexcloud.net/cloud-certs/CA.pem" --output-document yandex-kafka-production/app/ca.crt`
* Посмотреть в консоли Yandex Cloud хост для подключения к кластеру Kafka и создать файл `yandex-kafka-production/app/.env` со следующим содержимым (заменив имя хоста и пароль на свои):
```
PRODUCER_KAFKA_BOOTSTRAP_SERVERS=rc1a-96eh53qo92ss79ic.mdb.yandexcloud.net:9091
PRODUCER_SCHEMA_REGISTRY_SERVER=https://kafka-user:...@rc1a-96eh53qo92ss79ic.mdb.yandexcloud.net:443
PRODUCER_KAFKA_PASSWORD=...

CONSUMER_KAFKA_BOOTSTRAP_SERVERS=rc1a-96eh53qo92ss79ic.mdb.yandexcloud.net:9091
CONSUMER_SCHEMA_REGISTRY_SERVER=https://kafka-user:...@rc1a-96eh53qo92ss79ic.mdb.yandexcloud.net:443
CONSUMER_KAFKA_PASSWORD=...
```
* Выполнить `cd yandex-kafka-production/app`
* Выполнить `docker compose up -d`

## Тестирование

* Пошлём сообщения через producer: `for i in $(seq 1 100); do curl -vv -XPOST -d '{"text": "message"}' -H "Content-Type: application/json" localhost:9000/api/v1/kafka/send; done`
* Посмотрим логи контейнеров `docker compose logs -f`
* В логах продьюсера должны быть сообщения об успешной отправке
```
producer-1     | 2025-06-01 17:37:05.372 | INFO     | producer.endpoints.kafka:send_kafka:26 - "{\"text\":\"message\"}"
producer-1     | INFO:     172.19.0.1:58920 - "POST /api/v1/kafka/send HTTP/1.1" 200 OK
producer-1     | 2025-06-01 17:37:05.383 | INFO     | producer.endpoints.kafka:send_kafka:26 - "{\"text\":\"message\"}"
producer-1     | INFO:     172.19.0.1:58924 - "POST /api/v1/kafka/send HTTP/1.1" 200 OK
```
* В логах консьюмера должны быть сообщения об успешном получении
```
consumer-1     | 2025-06-01 17:37:11.034 | INFO     | src.consumer.service:process_message:16 - {"topic": "kafka-topic", "partition": 1, "offset": 0, "key": null, "message": {"text": "message"}, "timestamp": 1748799424732}
consumer-1     | 2025-06-01 17:37:21.020 | INFO     | src.consumer.service:process_message:16 - {"topic": "kafka-topic", "partition": 0, "offset": 0, "key": null, "message": {"text": "message"}, "timestamp": 1748799424743}
```
* Скриншот:

![logs](README_img/logs.png)

* Выполним `curl -k https://kafka-user:...@rc1a-96eh53qo92ss79ic.mdb.yandexcloud.net:443/subjects`

```
["kafka-topic-message"]
```

* Выполним `curl -k https://kafka-user:...@rc1a-96eh53qo92ss79ic.mdb.yandexcloud.net:443/subjects/kafka-topic-message/versions`

```
[1]
```

* Так как мы не используем файл схемы (producer автоматически его генерирует при помощи библиотеки pydantic-avro), извлечём схему с Schema Registry при помощи команды `curl -k https://kafka-user:...@rc1a-96eh53qo92ss79ic.mdb.yandexcloud.net:443/schemas`

```
[{"id":1,"schema":"{\"fields\":[{\"name\":\"text\",\"type\":\"string\"}],\"name\":\"Message\",\"namespace\":\"Message\",\"type\":\"record\"}","schemaType":"AVRO","subject":"kafka-topic-message","version":1}]
```