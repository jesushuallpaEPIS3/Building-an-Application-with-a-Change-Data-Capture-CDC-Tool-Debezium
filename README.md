# Building an Application with a Change Data Capture (CDC) Tool: Debezium

![JESUS ANTONIO HUALLPA MARON's photo](#)

**JESUS ANTONIO HUALLPA MARON**  
Dec 12, 2024 · 3 min read  

---

## Introduction

In today's world of real-time data applications, Change Data Capture (CDC) has become a key solution for synchronizing data across systems. CDC enables the identification and capture of changes made to a database, such as inserts, updates, and deletes, and propagates them to other systems or applications.

Debezium is an open-source tool that facilitates CDC implementation by capturing events directly from database transaction logs. In this article, we will explore how to build an application using Debezium to capture changes from a MySQL database and process them in real-time.

---

## Features of Debezium

- **Integration with Kafka**: Uses Apache Kafka as the transport mechanism for change events.
- **Compatibility with popular databases**: MySQL, PostgreSQL, MongoDB, Oracle, among others.
- **High availability**: Designed for distributed and resilient applications.
- **Real-time change detection**: Continuous monitoring of transaction logs.

---

## Initial Setup

### 1. Prerequisites
- Docker and Docker Compose installed.
- Basic knowledge of Apache Kafka and MySQL.
- Text editor or IDE (e.g., Visual Studio Code).

### 2. Environment Setup
We will create an environment using Docker Compose that includes:
- A MySQL instance.
- A Kafka cluster.
- The Debezium connector.

### Docker Compose File (`docker-compose.yml`):

```yaml
version: '3.8'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181

  kafka:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    environment:
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1

  mysql:
    image: mysql:latest
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: testdb

  debezium:
    image: debezium/connect:latest
    depends_on:
      - kafka
    environment:
      BOOTSTRAP_SERVERS: kafka:9092
      GROUP_ID: debezium-connector
      CONFIG_STORAGE_TOPIC: debezium_config
      OFFSET_STORAGE_TOPIC: debezium_offset
      STATUS_STORAGE_TOPIC: debezium_status
```
Run the environment with the command:

```yaml
docker-compose up -d
```
Building the Application
Step 1: Configure the Debezium Connector
Register the connector for MySQL by sending a POST request to Debezium:
```yaml
curl -X POST -H "Content-Type: application/json" \
  --data '{
    "name": "mysql-connector",
    "config": {
      "connector.class": "io.debezium.connector.mysql.MySqlConnector",
      "tasks.max": "1",
      "database.hostname": "mysql",
      "database.port": "3306",
      "database.user": "root",
      "database.password": "root",
      "database.server.id": "184054",
      "database.server.name": "dbserver1",
      "database.include.list": "testdb",
      "table.include.list": "testdb.users",
      "database.history.kafka.bootstrap.servers": "kafka:9092",
      "database.history.kafka.topic": "schema-changes.testdb"
    }
  }' \
  http://localhost:8083/connectors

```
Step 2: Capturing Events in Kafka
Use the Kafka client to consume events:
```yaml
kafka-console-consumer \
  --bootstrap-server kafka:9092 \
  --topic dbserver1.testdb.users \
  --from-beginning

```
Step 3: Processing Events in the Application
Develop an application (e.g., in Python) to process the events.

Sample Code (Python):
```yaml
from kafka import KafkaConsumer
import json

consumer = KafkaConsumer(
    'dbserver1.testdb.users',
    bootstrap_servers='localhost:9092',
    auto_offset_reset='earliest',
    enable_auto_commit=True,
    group_id='my-group',
    value_deserializer=lambda x: json.loads(x.decode('utf-8'))
)

for message in consumer:
    event = message.value
    print(f"Received event: {event}")


```
Conclusion
Debezium simplifies the implementation of Change Data Capture by providing direct integration with database transaction logs. The combination of Debezium and Apache Kafka enables the development of highly scalable and resilient applications for real-time data processing. This approach is ideal for use cases such as database replication, system synchronization, and change monitoring.
