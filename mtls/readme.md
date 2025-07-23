# mTLS Setup:

# Create CA

```
openssl req -new -x509 -keyout ca-key -out ca-cert -days 3650
```

# For Kafka Server:

```
keytool -keystore kafka.truststore.jks -alias ca-cert -import -file ca-cert
keytool -keystore kafka.keystore.jks -alias kafka -validity 3650 -genkey -keyalg RSA -ext SAN=dns:localhost
keytool -keystore kafka.keystore.jks -alias kafka -certreq -file ca-request-kafka
openssl x509 -req -CA ca-cert -CAkey ca-key -in ca-request-kafka -out ca-signed-kafka -days 3650 -CAcreateserial
keytool -keystore kafka.keystore.jks -alias ca-cert -import -file ca-cert
keytool -keystore kafka.keystore.jks -alias kafka -import -file ca-signed-kafka
```
# For Kafka Client:

```
keytool -keystore client.truststore.jks -alias ca-cert -import -file ca-cert
keytool -keystore client.keystore.jks -alias client -validity 3650 -genkey -keyalg RSA -ext SAN=dns:localhost
keytool -keystore client.keystore.jks -alias client -certreq -file ca-request-client
openssl x509 -req -CA ca-cert -CAkey ca-key -in ca-request-client -out ca-signed-client -days 3650 -CAcreateserial
keytool -keystore client.keystore.jks -alias ca-cert -import -file ca-cert
keytool -keystore client.keystore.jks -alias client -import -file ca-signed-client
```
# Server Properties:

```
ssl.client.auth=required
security.protocol=SSL
ssl.protocol=TLSv1.2
ssl.truststore.location=/Users/apple/Documents/kafka_2.12-3.9.1/mtls/kafka.truststore.jks
ssl.truststore.password=qatest
ssl.keystore.location=/Users/apple/Documents/kafka_2.12-3.9.1/mtls/kafka.keystore.jks
ssl.keystore.password=qatest
ssl.key.password=qatest
```

# Client Properties:

```
security.protocol=SSL
ssl.truststore.location=/Users/apple/Documents/kafka_2.12-3.9.1/mtls/client.truststore.jks
ssl.truststore.password=qatest
ssl.keystore.location=/Users/apple/Documents/kafka_2.12-3.9.1/mtls/client.keystore.jks
ssl.keystore.password=qatest
ssl.key.password=qatest
```

# Start Controller:

```
./bin/kafka-server-start.sh mtls/config/controller.properties 
```

# Start Broker:

```
./bin/kafka-server-start.sh mtls/config/broker-0.properties
```

# Create Topic:

```
./bin/kafka-topics.sh --create --topic ssltopic --bootstrap-server localhost:9070 --partitions 2 --replication-factor 1
```

# Secured Kafka Producer:

```
KAFKA_OPTS="-Djavax.net.debug=ssl" \                                              
./bin/kafka-console-producer.sh \
--bootstrap-server localhost:9097 \
--topic ssltopic \
--producer.config mtls/config/producer.properties
```

# Consume Message:

```
./bin/kafka-console-consumer.sh --topic ssltopic --bootstrap-server localhost:9070
```