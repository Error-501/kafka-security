# SASL_PLAIN Setup:

# JAAS Configuration

Create the JAAS configuration file for server authentication:

```
# sasl_plain/acl/kafka_server_jaas.conf
KafkaServer {
  org.apache.kafka.common.security.plain.PlainLoginModule required
  username="masteradmin"
  password="admin"
  user_admin="admin-qatest"
  user_alice="alice-qatest";
};
```

# Server Properties:

```
# Broker configuration
listeners=SASL_PLAINTEXT://localhost:9099,PLAINTEXT://localhost:9070
inter.broker.listener.name=SASL_PLAINTEXT
advertised.listeners=SASL_PLAINTEXT://localhost:9099,PLAINTEXT://localhost:9070
listener.security.protocol.map=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL
sasl.mechanism.inter.broker.protocol=PLAIN
sasl.enabled.mechanisms=PLAIN
```

# Client Properties:

## Producer Properties:
```
bootstrap.servers=localhost:9099
security.protocol=SASL_PLAINTEXT
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
  username="alice" \
  password="alice-qatest";
```

## Consumer Properties:
```
bootstrap.servers=localhost:9099
group.id=test-consumer-group
security.protocol=SASL_PLAINTEXT
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
  username="alice" \
  password="alice-qatest";
```

# Start Controller:

```
./bin/kafka-server-start.sh sasl_plain/config/controller.properties 
```

# Start Broker:

Set the JAAS configuration and start the broker:

```
export KAFKA_OPTS="-Djava.security.auth.login.config=/Users/apple/Documents/kafka_2.12-3.9.1/sasl_plain/acl/kafka_server_jaas.conf"
./bin/kafka-server-start.sh sasl_plain/config/broker-0.properties
```

# Create Topic:

```
./bin/kafka-topics.sh --create --topic sasltopic --bootstrap-server localhost:9070 --partitions 2 --replication-factor 1
```

# Secured Kafka Producer:

```
./bin/kafka-console-producer.sh \
--bootstrap-server localhost:9099 \
--topic sasltopic \
--producer.config sasl_plain/config/producer.properties
```

# Consume Message:

```
./bin/kafka-console-consumer.sh \
--topic sasltopic \
--bootstrap-server localhost:9099 \
--consumer.config sasl_plain/config/consumer.properties
```

# User Management:

The JAAS configuration defines users and their passwords:
- **admin**: admin-qatest (server admin user)
- **alice**: alice-qatest (client user for producer/consumer)
- **waction**: w@ct10n (broker inter-communication user)

To add more users, update the `kafka_server_jaas.conf` file with additional `user_<username>="<password>"` entries.
