# Kafka Command Hepler

- create topic `bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic mmxtest`
- get topic list `bin/kafka-topics.sh --list --zookeeper localhost:2181`
- send message `bin/kafka-console-producer.sh --broker-list localhost:9092 --topic mmxtest`
- receive message `bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic mmxtest --from-beginning`


> Kafka Source 
> https://kafka.apache.org/quickstart
