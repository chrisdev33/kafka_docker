Kafka
========
Kafka on your desktop
--------
- - - -

## 1. Lancement du cluster

2 docker-compose avec 2 options pour les IHM de monitoring Kafka :
* kafka-ui
* kafdrop

Le docker-compose lance un cluster Kafka composé de plusieurs containers :
* 1 Zookeeper
* 3 brokers
* 1 Schema Registry
* 1 IHM de monitoring (kafka-ui ou kafdrop)


### 1.1 Lancement avec Kafka-ui

> Lancement du cluster :

```shell
cd with_kafka-ui
docker-compose up -d
```

> Ne pas oublier de supprimer les containers si lancement de docker-compose avec kafdrop :

```shell
docker rm -f schema-registry kafka_broker_1 kafka_broker_3 kafka_broker_2 zookeeper
```

> L'url de kafka-ui est accessible via : http://localhost:8080/

> Arrêt du cluster :

```shell
docker-compose down
```

### 1.2 Lancement avec Kadrop

> Lancement du cluster :

```shell
cd with_kafdrop
docker-compose up -d
```

> Ne pas oublier de supprimer les containers si lancement de docker-compose avec kafka-ui :

```shell
docker rm -f schema-registry kafka_broker_1 kafka_broker_3 kafka_broker_2 zookeeper
```

> Vérifier que le cluster est up :

```shell
docker-compose ps
```

> L'url de kafka-ui est accessible via : http://localhost:9000/

> Arrêt du cluster :

```shell
docker-compose down
```

## 2. Exemples de commandes

### 2.1 Commandes topic

> Lister les topics

```shell
docker-compose exec kafka_broker_1 \
kafka-topics --list \
--bootstrap-server kafka_broker_1:9092
```

> Créer un topic :

```shell
docker-compose exec kafka_broker_1 \
kafka-topics --create --topic orders-avro \
--partitions 2 --replication-factor 3 \
--bootstrap-server kafka_broker_1:9092
```

### 2.2 Consumer et producer avec schema registry

> Dans le répertoire du docker-compose, le fichier orders-avro-schema.json contient le schéma :

```json
{
   "type": "record",
   "namespace": "io.confluent.tutorial",
   "name": "OrderDetail",
   "fields": [
      {"name": "number", "type": "long", "doc": "The order number."},
      {"name": "shipping_address", "type": "string", "doc": "The shipping address."},
      {"name": "subtotal", "type": "double", "doc": "The amount without shipping cost and tax."},
      {"name": "shipping_cost", "type": "double", "doc": "The shipping cost."},
      {"name": "tax", "type": "double", "doc": "The applicable tax."},
      {"name": "grand_total", "type": "double", "doc": "The order grand total ."}
   ]
}
```

> Lancement du consumer

```shell
docker-compose exec schema-registry bash

kafka-avro-console-consumer \
  --topic orders-avro \
  --bootstrap-server kafka_broker_1:9092 \
  --property schema.registry.url=http://localhost:8081
```

> Lancement du producer

> Ouvrir un autre shell pour lancer le producer

```shell
docker-compose exec schema-registry bash

kafka-avro-console-producer \
  --topic orders-avro \
  --bootstrap-server kafka_broker_1:9092 \
  --property schema.registry.url=http://localhost:8081 \
  --property value.schema="$(< /etc/tutorial/orders-avro-schema.json)"
```

> Copier/coller ces lignes pour produire un event

```shell
{"number":1,"shipping_address":"ABC Sesame Street,Wichita, KS. 12345","subtotal":110.00,"tax":10.00,"grand_total":120.00,"shipping_cost":0.00}
{"number":2,"shipping_address":"123 Cross Street,Irving, CA. 12345","subtotal":5.00,"tax":0.53,"grand_total":6.53,"shipping_cost":1.00}
{"number":3,"shipping_address":"5014  Pinnickinick Street, Portland, WA. 97205","subtotal":93.45,"tax":9.34,"grand_total":102.79,"shipping_cost":0.00}
{"number":4,"shipping_address":"4082 Elmwood Avenue, Tempe, AX. 85281","subtotal":50.00,"tax":1.00,"grand_total":51.00,"shipping_cost":0.00}
{"number":5,"shipping_address":"123 Cross Street,Irving, CA. 12345","subtotal":33.00,"tax":3.33,"grand_total":38.33,"shipping_cost":2.00}
```

Dans le shell du consumer, les events doivent s'afficher

## 3. Référence

* Kafka documentation : https://kafka.apache.org/documentation/
* UI Tools for monitoring Kafka : https://towardsdatascience.com/overview-of-ui-tools-for-monitoring-and-management-of-apache-kafka-clusters-8c383f897e80
* kafka-ui : https://github.com/provectus/kafka-ui
* Explication Schema Registry : https://blog.ippon.fr/2019/11/18/confluent-schema-registry-un-premier-pas-vers-la-gouvernance-des-donnees/#:~:text=Schema%20Registry%20%26%20Kafka,envoy%C3%A9s%20dans%20les%20topics%20Kafka.
* https://docs.confluent.io/platform/current/overview.html#confluent-platform





