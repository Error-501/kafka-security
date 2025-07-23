# Basic Kraft Setup:

# Create cluster ID:

```
bin/kafka-storage.sh random-uuid
```

# Set log.dirs in config/kraft/server.properties

# Create log directories for each node:

```
mkdir -p /Users/apple/Documents/kafka_2.12-3.9.1/data/controller
mkdir -p /Users/apple/Documents/kafka_2.12-3.9.1/data/broker-0
```

# Format the log directory for each node:

```
bin/kafka-storage.sh format --standalone -t <cluster-id> -c config/kraft/controller.properties
bin/kafka-storage.sh format -t <cluster-id> -c config/kraft/broker-0.properties
```

# Start the server:

```
bin/kafka-server-start.sh config/kraft/controller.properties
```

# Start the broker:

```
bin/kafka-server-start.sh config/kraft/broker-0.properties
```

# Create Topic:

```
bin/kafka-topics.sh --create --topic sslone --bootstrap-server localhost:9092 --partitions 2 --replication-factor 1
```
