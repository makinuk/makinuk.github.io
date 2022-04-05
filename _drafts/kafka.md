# Kafka Command Hepler

- create topic `bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic mmxtest`
- get topic list `bin/kafka-topics.sh --list --zookeeper localhost:2181`
- send message `bin/kafka-console-producer.sh --broker-list localhost:9092 --topic mmxtest`
- receive message `bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic mmxtest --from-beginning`

> Kafka Source 
> https://kafka.apache.org/quickstart

# Kafka Installation : 
## Zookeeper : 
  - `dataDir=/path/to/zookeeper` change data directory 
  - define server list as below
    ```
    4lw.commands.whitelist=*
    
    #tickTime=2000
    initLimit=5
    syncLimit=2
    
    server.0=HOST/IPADDR:2888:3888
    server.1=HOST/IPADDR:2888:3888
    server.2=HOST/IPADDR:2888:3888
    ...
    ``` 
  - create myid file to all node with server id exm for server 0 `echo '0' > /path/to/zookeeper/myid`
  - add zookeeper as service 
   ```
[Unit]
Requires=network.target remote-fs.target
After=network.target remote-fs.target

[Service]
Type=simple
User=kafka
ExecStart=/opt/kafka/bin/zookeeper-server-start.sh /opt/kafka/config/zookeeper.properties
ExecStop=/opt/kafka/bin/zookeeper-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
```
## Kafka : 
- `vi /opt/kafka/config/server.properties` open config file
-  change broker id `broker.id=0` The id of the broker. This must be set to a unique integer for each broker.
-  `zookeeper.connect=HOST/IPADDR:2181,HOST/IPADDR:2181,HOST/IPADDR:2181`  This is a comma separated host:port pairs, each corresponding to a zk server
-  `log.dirs=/mnt/dmc_data/kafka-logs`  A comma separated list of directories under which to store log files
-  delete.topic.enable = true
-  add kafka as a service 
```
[Unit]
Requires=zookeeper.service
After=zookeeper.service

[Service]
#Environment=JMX_PORT=9989
#Environment=KAFKA_JMX_OPTS="-Dcom.sun.management.jmxremote=true -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -#Djava.rmi.server.hostname=JMXREmote HOST NAME"
Type=simple
User=kafka
ExecStart=/opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties
ExecStop=/opt/kafka/bin/kafka-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
```

