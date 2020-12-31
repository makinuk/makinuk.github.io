---
layout: post	
title: Kafka 2.7.0 Installatin on CentOS 8 	
published: true	
categories: [kafka]	
date: 2021-01-01 09:00
excerpt: | 
     Kafka instalation on CentOS 8
     
---

 
## Introduction

Apache Kafka is a popular distributed message broker designed to efficiently handle large volumes of real-time data. 
A Kafka cluster is not only highly scalable and fault-tolerant, but it also has a much higher throughput compared to other message brokers such as ActiveMQ and RabbitMQ. 
Though it is generally used as a publish/subscribe messaging system, a lot of organizations also use it for log aggregation because it offers persistent storage for published messages.

A publish/subscribe messaging system allows one or more producers to publish messages without considering the number of consumers or how they will process the messages. 
Subscribed clients are notified automatically about updates and the creation of new messages. 
This system is more efficient and scalable than systems where clients poll periodically to determine if new messages are available.

In this tutorial, you will install and use Apache Kafka 2.7.0 on CentOS 8.

## Prerequisites

To follow along, you will need:
- One CentOS 8 server and a non-root user with sudo privileges.
- At least 4GB of RAM on the server. Installations without this amount of RAM may cause the Kafka service to fail, with the Java virtual machine (JVM) throwing an “Out Of Memory” exception during startup.
- OpenJDK 8 installed on your server. To install this version, follow these instructions on installing specific versions of OpenJDK. Kafka is written in Java, so it requires a JVM; however, its startup shell script has a version detection bug that causes it to fail to start with JVM versions above 8.

## Step 1 — Creating a User for Kafka and Firewall rules

### Firewall Rules
As mentioned above we have to open several ports to allow clients to connect to our Kafka cluster and to allow the nodes to communicate with each other. In CentOS 8 we have to add the corresponding firewall rules.

Let's start with the firewall rule for ZooKeeper. We have to open ports 2888, 3888 and 2181.
```bash
root@bari01$ vi /etc/firewalld/services/zooKeeper.xml
```

We have to add the following content to the zooKeeper.xml file.
```
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>ZooKeeper</short>
  <description>Firewall rule for ZooKeeper ports</description>
  <port protocol="tcp" port="2888"/>
  <port protocol="tcp" port="3888"/>
  <port protocol="tcp" port="2181"/>
</service>
```

For Kafka, we have to open port 9092.
```bash
$ vi /etc/firewalld/services/kafka.xml
```

We have to add the following content to the kafka.xml file.
```
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>Kafka</short>
  <description>Firewall rule for Kafka port</description>
  <port protocol="tcp" port="9092"/>
</service>
```

Now we can activate the new firewall rules. Let's first restart the firewalld service to enforce that all existing service specifications are reloaded.
```bash
$ systemctl restart firewalld
```
We can now permanently add the firewall rules for ZooKeeper and Kafka.
```bash
$ firewall-cmd --permanent --add-service=zooKeeper
$ firewall-cmd --permanent --add-service=kafka
```

After activating the rules we have to restart firewalld.
```bash
$ systemctl restart firewalld
```

### Users
Since Kafka can handle requests over a network, you should create a dedicated user for it. This minimizes damage to your CentOS machine should the Kafka server be compromised. We will create a dedicated kafka user in this step, but you should create a different non-root user to perform other tasks on this server once you have finished setting up Kafka.

Logged in as your non-root sudo user, create a user called kafka with the useradd command:
```bash
$ sudo useradd kafka -m
```
The -m flag ensures that a home directory will be created for the user. This home directory, /home/kafka, will act as our workspace directory for executing commands in the sections below.

Set the password using `passwd`:
```bash
$ sudo passwd kafka
```

Add the kafka user to the wheel group with the adduser command, so that it has the privileges required to install Kafka’s dependencies:
```bash
$ sudo usermod -aG wheel kafka
```

Your kafka user is now ready. Log into this account using `su`
```bash
$ su -l kafka
```

Now that we’ve created the Kafka-specific user, we can move on to downloading and extracting the Kafka binaries.

## Step 2 — Downloading and Extracting the Kafka Binaries
Let’s download and extract the Kafka binaries into dedicated folders in our kafka user’s home directory.

To start, create a directory in `/home/kafka` called `Downloads` to store your downloads:
```bash 
`$ mkdir ~/Downloads
```

Use curl to download the *Kafka* binaries:
```bash 
$ curl "https://downloads.apache.org/kafka/2.7.0/kafka_2.13-2.7.0.tgz" -o ~/Downloads/kafka.tgz
```
Create a directory called `kafka` and change to this directory. This will be the base directory of the Kafka installation:
```bash
$ mkdir ~/kafka && cd ~/kafka
```
Extract the archive you downloaded using the `tar` command:
```bash
$ tar -xvzf ~/Downloads/kafka.tgz --strip 1
```

We specify the `--strip 1` flag to ensure that the archive’s contents are extracted in `~/kafka/` itself and not in another directory (such as `~/kafka/kafka_2.11-2.1.1/`) inside of it.

Now that we’ve downloaded and extracted the binaries successfully, we can move on configuring to Kafka to allow for topic deletion.

## Step 3 — Configuring the Kafka Server

Kafka’s default behavior will not allow us to delete a topic, the category, group, or feed name to which messages can be published. To modify this, let’s edit the configuration file.

Kafka’s configuration options are specified in `server.properties`. Open this file with vi or your favorite editor:
```bash 
$ vi ~/kafka/config/server.properties
```

Let’s add a setting that will allow us to delete Kafka topics. Press i to insert text, and add the following to the bottom of the file:
```bash 
$ delete.topic.enable = true
```

When you are finished, press ESC to exit insert mode and :wq to write the changes to the file and quit. Now that we’ve configured Kafka, we can move on to creating systemd unit files for running and enabling it on startup.

## Step 4 — Creating Systemd Unit Files and Starting the Kafka Server

In this section, we will create systemd unit files for the Kafka service. This will help us perform common service actions such as starting, stopping, and restarting Kafka in a manner consistent with other Linux services.

Zookeeper is a service that Kafka uses to manage its cluster state and configurations. It is commonly used in many distributed systems as an integral component. If you would like to know more about it, visit the official Zookeeper docs.

Create the unit file for zookeeper:
```bash
$ sudo vi /etc/systemd/system/zookeeper.service
```
Enter the following unit definition into the file:
```
[Unit]
Requires=network.target remote-fs.target
After=network.target remote-fs.target

[Service]
Type=simple
User=kafka
ExecStart=/home/kafka/kafka/bin/zookeeper-server-start.sh /home/kafka/kafka/config/zookeeper.properties
ExecStop=/home/kafka/kafka/bin/zookeeper-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
```
The `[Unit]` section specifies that Zookeeper requires networking and the filesystem to be ready before it can start.

The `[Service]` section specifies that systemd should use the zookeeper-server-start.sh and zookeeper-server-stop.sh shell files for starting and stopping the service. It also specifies that Zookeeper should be restarted automatically if it exits abnormally.

Save and close the file when you are finished editing.

Next, create the systemd service file for kafka:

```bash
$ sudo vi /etc/systemd/system/kafka.service
```

Enter the following unit definition into the file:
```
[Unit]
Requires=zookeeper.service
After=zookeeper.service

[Service]
Type=simple
User=kafka
ExecStart=/bin/sh -c '/home/kafka/kafka/bin/kafka-server-start.sh /home/kafka/kafka/config/server.properties > /home/kafka/kafka/kafka.log 2>&1'
ExecStop=/home/kafka/kafka/bin/kafka-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
```

The `[Unit]` section specifies that this unit file depends on zookeeper.service. This will ensure that zookeeper gets started automatically when the kafa service starts.

The `[Service]` section specifies that systemd should use the kafka-server-start.sh and kafka-server-stop.sh shell files for starting and stopping the service. It also specifies that Kafka should be restarted automatically if it exits abnormally.

Save and close the file when you are finished editing.

Now that the units have been defined, start Kafka with the following command:

```bash
$ sudo systemctl start kafka
```

To ensure that the server has started successfully, check the journal logs for the kafka unit:
```bash 
$ journalctl -u kafka
```

You now have a Kafka server listening on port `9092`

While we have started the kafka service, if we were to reboot our server, it would not be started automatically. To enable kafka on server boot, run:

```bash
$ sudo systemctl enable kafka
```
Now that we’ve started and enabled the services, let’s check the installation.

## Step 5 — Testing the Installation

Let’s publish and consume a “Hello World” message to make sure the Kafka server is behaving correctly. Publishing messages in Kafka requires:

- **A producer**, which enables the publication of records and data to topics.
- **A consumer**, which reads messages and data from topics.

First, create a topic named `TutorialTopic` by typing:
```bash
$ ~/kafka/bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic TutorialTopic
```

You will see the following output:
```bash

Created topic TutorialTopic.
```

You can create a producer from the command line using the kafka-console-producer.sh script. It expects the Kafka server’s hostname, port, and a topic name as arguments.

Publish the string "Hello, World" to the TutorialTopic topic by typing:
```bash
$ echo "Hello, World" | ~/kafka/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic TutorialTopic > /dev/null
```

Next, you can create a Kafka consumer using the `kafka-console-consumer.sh` script. It expects the ZooKeeper server’s hostname and port, along with a topic name as arguments.

The following command consumes messages from TutorialTopic. Note the use of the --from-beginning flag, which allows the consumption of messages that were published before the consumer was started

```bash 
$ ~/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic TutorialTopic --from-beginning
```

If there are no configuration issues, you should see `Hello, World` in your terminal:
```
Hello, World
```

The script will continue to run, waiting for more messages to be published to the topic. Feel free to open a new terminal and start a producer to publish a few more messages. You should be able to see them all in the consumer’s output.

When you are done testing, press `CTRL+C` to stop the consumer script. Now that we have tested the installation, let’s move on to installing KafkaT.

## Step 6 — Installing KafkaT (Optional)
[KafkaT](https://github.com/airbnb/kafkat) is a tool from Airbnb that makes it easier for you to view details about your Kafka cluster and perform certain administrative tasks from the command line. Because it is a Ruby gem, you will need Ruby to use it. You will also need ruby-devel and build-related packages such as make and gcc to be able to build the other gems it depends on. Install them using yum
```bash
$ sudo yum install ruby ruby-devel make gcc patch
``` 

You can now install KafkaT using the gem command:
```bash
$ sudo gem install kafkat
```

KafkaT uses .kafkatcfg as the configuration file to determine the installation and log directories of your Kafka server. It should also have an entry pointing KafkaT to your ZooKeeper instance.

Create a new file called `.kafkatcfg`:
```
$ vi ~/.kafkatcfg
```
Add the following lines to specify the required information about your Kafka server and Zookeeper instance:

```
{
  "kafka_path": "~/kafka",
  "log_path": "/tmp/kafka-logs",
  "zk_path": "localhost:2181"
}
```

Save and close the file when you are finished editing.

You are now ready to use KafkaT. For a start, here’s how you would use it to view details about all Kafka partitions:
```bash 
$ kafkat partitions
```

You will see the following output:
```
Topic                 Partition   Leader      Replicas        ISRs    
TutorialTopic         0             0         [0]             [0]
__consumer_offsets    0             0         [0]                           [0]
...
...
```

You will see TutorialTopic, as well as __consumer_offsets, an internal topic used by Kafka for storing client-related information. You can safely ignore lines starting with __consumer_offsets.

To learn more about KafkaT, refer to its [GitHub repository](https://github.com/airbnb/kafkat).

## Step 7 — Setting Up a Multi-Node Cluster (Optional)

If you want to create a multi-broker cluster using more CentOS 8 machines, you should repeat Step 1, Step 4, and Step 5 on each of the new machines. Additionally, you should make the following changes in the server.properties file for each:

- The value of the `broker.id` property should be changed such that it is unique throughout the cluster. This property uniquely identifies each server in the cluster and can have any string as its value. For example, "server1", "server2", etc.
- The value of the zookeeper.connect property should be changed such that all nodes point to the same ZooKeeper instance. This property specifies the Zookeeper instance’s address and follows the <HOSTNAME/IP_ADDRESS>:<PORT> format. For example, "203.0.113.0:2181", "203.0.113.1:2181" etc.

If you want to have multiple ZooKeeper instances for your cluster, the value of the zookeeper.connect property on each node should be an identical, comma-separated string listing the IP addresses and port numbers of all the ZooKeeper instances.

--- 
### if clustur not working check this :
> `ERROR Fatal error during KafkaServer startup. Prepare to shutdown (kafka.server.KafkaServer)
kafka.common.InconsistentClusterIdException: The Cluster ID jvrDMiqjSWmPEMoAPD9iMQ doesn't match stored clusterId Some(DrtQg1zUTAW4VoYZjb_F7A) in meta.properties. The broker is trying to join the wrong cluster. Configured zookeeper.connect may be wrong.`

> The Kafka data directory (check config/server.properties for log.dirs property, it defaults to /tmp/kafka-logs) contains a file called meta.properties. It contains the cluster ID. Which should have matched the ID registered to ZK. Either edit the file to match ZK, edit ZK to match the file, or delete the file (it contains the cluster id and the broker id, the first is currently broken and the second is in the config file normally). After this minor surgery, Kafka will start with all your existing data, since you didn't delete any data file.
> 
> Like this: `mv /tmp/kafka-logs/meta.properties /tmp/kafka-logs/meta.properties_old`

## Step 8 — Restricting the Kafka User
Now that all of the installations are done, you can remove the kafka user’s admin privileges. Before you do so, log out and log back in as any other non-root sudo user. If you are still running the same shell session you started this tutorial with, simply type exit.

Remove the kafka user from the sudo group:
```bash
sudo gpasswd -d kafka wheel
```

To further improve your Kafka server’s security, lock the kafka user’s password using the `passwd` command. This makes sure that nobody can directly log into the server using this account:
```bash
$ sudo passwd kafka -l
```

At this point, only root or a sudo user can log in as kafka by typing in the following command:
```bash
$ sudo su - kafka
```

In the future, if you want to unlock it, use passwd with the -u option:
```bash
$ sudo passwd kafka -u
```
You have now successfully restricted the kafka user’s admin privileges.

[//]: <> (https://www.digitalocean.com/community/tutorials/how-to-install-apache-kafka-on-centos-7)


