# Hello PUBSUB team! 

In this task, I will install Confluent Platform with Docker, verify it is working by creating a topic, sending and consuming some messages, and finally removing those messages from the topic.

## Objectives 

1. Install Confluent Platform
2. Create Kafka topic
3. Produce messages into the topic
4. Consume those messages from the topic
5. Remove all the messages from the topic

## Procedure 

For this activity, I am using my home lab, which runs CentOS Stream 9 on VirtualBox. As a prerequisite, Docker has already been installed on the machine.  

1. Download the Docker Compose file and start Confluent Platform:

```
# wget https://raw.githubusercontent.com/confluentinc/cp-all-in-one/7.8.0-post/cp-all-in-one-kraft/docker-compose.yml
...output omitted...
# docker compose up -d
[+] Running 8/8
 ✔ Container broker           Running                                                                                                                    0.0s 
 ✔ Container schema-registry  Running                                                                                                                    0.0s 
 ✔ Container connect          Running                                                                                                                    0.0s 
 ✔ Container ksqldb-server    Running                                                                                                                    0.0s 
 ✔ Container ksql-datagen     Running                                                                                                                    0.0s 
 ✔ Container rest-proxy       Running                                                                                                                    0.0s 
 ✔ Container control-center   Running                                                                                                                    0.0s 
 ✔ Container ksqldb-cli       Running                                                                                                                    0.0s
```

We can verify that the services are up and running:

```
# docker compose ps
NAME              IMAGE                                             COMMAND                  SERVICE           CREATED              STATUS              PORTS
broker            confluentinc/cp-kafka:7.8.0                       "/etc/confluent/dock…"   broker            About a minute ago   Up About a minute   0.0.0.0:9092->9092/tcp, :::9092->9092/tcp, 0.0.0.0:9101->9101/tcp, :::9101->9101/tcp
connect           cnfldemos/cp-server-connect-datagen:0.6.4-7.6.0   "/etc/confluent/dock…"   connect           About a minute ago   Up About a minute   0.0.0.0:8083->8083/tcp, :::8083->8083/tcp, 9092/tcp
control-center    confluentinc/cp-enterprise-control-center:7.8.0   "/etc/confluent/dock…"   control-center    About a minute ago   Up 55 seconds       0.0.0.0:9021->9021/tcp, :::9021->9021/tcp
ksql-datagen      confluentinc/ksqldb-examples:7.8.0                "bash -c 'echo Waiti…"   ksql-datagen      About a minute ago   Up 55 seconds       
ksqldb-cli        confluentinc/cp-ksqldb-cli:7.8.0                  "/bin/sh"                ksqldb-cli        About a minute ago   Up 55 seconds       
ksqldb-server     confluentinc/cp-ksqldb-server:7.8.0               "/etc/confluent/dock…"   ksqldb-server     About a minute ago   Up 59 seconds       0.0.0.0:8088->8088/tcp, :::8088->8088/tcp
rest-proxy        confluentinc/cp-kafka-rest:7.8.0                  "/etc/confluent/dock…"   rest-proxy        About a minute ago   Up About a minute   0.0.0.0:8082->8082/tcp, :::8082->8082/tcp
schema-registry   confluentinc/cp-schema-registry:7.8.0             "/etc/confluent/dock…"   schema-registry   About a minute ago   Up About a minute   0.0.0.0:8081->8081/tcp, :::8081->8081/tcp
```

2. Initially, I created two topics using the Confluent Control Center web-based UI. However, I later shifted my focus to using the CLI for topic creation. Below, I will describe the workflow used to accomplish this.

First, I will enter the Kafka broker container to interact with it:

```
# docker exec -it broker bash
```

Once inside the container, I will use the ```kafka-topics``` command to create a new topic named _hithere_:

```
[appuser@broker ~]$ kafka-topics --bootstrap-server :9092 --create  --partitions 1 --topic hithere
Created topic hithere.
```

To confirm the topic creation, I will list all available topics:

```
[appuser@broker ~]$ kafka-topics --bootstrap-server :9092 --list
...output omitted...
hithere
```

3. To produce messages to the _hithere_ topic, I will use the ```kafka-console-producer``` command:

```
[appuser@broker ~]$ kafka-console-producer --bootstrap-server :9092 --topic hithere
>Hey to you         
>do you see me   
>hello
```



4. To consume the messages, I will open second terminal and use ```kafka-console-consumer``` command:

```
[appuser@broker ~]$ kafka-console-consumer --bootstrap-server localhost:9092 --topic hithere --from-beginning
Hey to you 
do you see me
hello
^CProcessed a total of 3 messages
[appuser@broker ~]$ 

```

5. Lastly, I will remove all messages from the topic. As mentioned in the task description, I tested two methods for this: deleting the topic entirely and adjusting the ```retention.ms``` setting. For this, I used both approaches to ensure all messages were removed, depending on the requirements of the task.

