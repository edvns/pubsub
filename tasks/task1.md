# Hello PUBSUB team! 

In this task, I will install Confluent Platform with Docker, verify it is working by creating a topic, sending and consuming some messages, and finally removing those messages from the topic.

## Objectives 

1. Install Confluent Platform
2. Create Kafka topic
3. Produce messages into the topic
4. Consume those messages from the topic
5. Remove all the messages from the topic

## Workflow 

For this activity, I am using my home lab with CentOS Stream 9 running on VirtualBox. As a pre-requisite, I've installed Docker on the machine.  

1. As a first step, I've downloaded docker compose file and started Confluen Platform as shown below:

```
wget 
docker compose up -d
```

2. 
