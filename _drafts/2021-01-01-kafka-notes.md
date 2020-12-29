---
layout: post	
title: Kafka 2.7.0 Installatin on CentOS 8 	
published: true	
categories: [kafka]	
date: 2021-01-01 09:00
excerpt: | 
     export file list as Json via jq
     
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
- One CentOS 7 server and a non-root user with sudo privileges.
- At least 4GB of RAM on the server. Installations without this amount of RAM may cause the Kafka service to fail, with the Java virtual machine (JVM) throwing an “Out Of Memory” exception during startup.
- OpenJDK 8 installed on your server. To install this version, follow these instructions on installing specific versions of OpenJDK. Kafka is written in Java, so it requires a JVM; however, its startup shell script has a version detection bug that causes it to fail to start with JVM versions above 8.

## Step 1 — Creating a User for Kafka

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
mkdir ~/kafka && cd ~/kafka
```
Extract the archive you downloaded using the `tar` command:
```bash
$ tar -xvzf ~/Downloads/kafka.tgz --strip 1
```

We specify the `--strip 1` flag to ensure that the archive’s contents are extracted in `~/kafka/` itself and not in another directory (such as `~/kafka/kafka_2.11-2.1.1/`) inside of it.

Now that we’ve downloaded and extracted the binaries successfully, we can move on configuring to Kafka to allow for topic deletion.

## Step 3 — Configuring the Kafka Server
https://www.digitalocean.com/community/tutorials/how-to-install-apache-kafka-on-centos-7


