# Trial task 2

In this task, I will work with the Confluent Kafka platform to produce and consume messages based on the Credit Card Approval Prediction dataset. The goal is to produce messages into a Kafka topic, ensure data deduplication, and then store deduplicated data into another topic.

## Objectives 

1. Set up a Kafka producer in Python that reads from the dataset and produces messages into a Kafka topic.
2. Deduplicate the messages and produce them into a new topic.

## Procedure 

For this activity, I am using a Credit Card Approval Prediction dataset named "application_record.csv". 

1. We start the activity by setting up Kafka producer.

First, I will download Confluent Kafka Python library:

```
pip install confluent_kafka
```

Below is python that we are going to use to produce data into topic

```
from confluent_kafka import Producer
import csv
import json

# Kafka producer configuration
conf = {
    'bootstrap.servers': 'localhost:9092'  # Kafka broker address
}

# Create a producer instance
producer = Producer(conf)

# Callback function to handle delivery report (success or failure)
def delivery_report(err, msg):
    if err is not None:
        print(f"Message delivery failed: {err}")
    else:
        print(f"Message delivered to {msg.topic()} [{msg.partition()}] at offset {msg.offset()}")

# Path to the CSV file 
csv_file_path = 'application_record.csv'
topic = 'credit_card_topic'  # Kafka topic name 

# Function to produce messages
def produce_messages():
    with open(csv_file_path, mode='r') as csvfile:
        reader = csv.DictReader(csvfile)  # Reading CSV as dictionaries (using column names as keys)

        # Limit to the first 20 rows
        rows_to_produce = list(reader)[:20]  # Get the first 20 rows

        # Produce each row twice (duplicate messages for testing duplicates)
        for row in rows_to_produce:
            # Get the "ID" field as the Kafka message key (ID is a number, we convert it to a string)
            message_key = str(row['ID'])  # Convert ID to string for Kafka key

            # Create the message value (the rest of the row data)
            message_value = {key: value for key, value in row.items() if key != 'ID'}  # Exclude 'ID' from the value

            # Produce the first message
            producer.produce(
                topic,  
                key=message_key,  # Setting "ID" as the key of the message
                value=json.dumps(message_value),  # The rest of the row to JSON format
                callback=delivery_report  # Callback to handle success/failure of the delivery
            )

            # Produce the second (duplicate) message
            producer.produce(
                topic,
                key=message_key,
                value=json.dumps(message_value),
                callback=delivery_report
            )

    # Ensure all messages are delivered before closing
    producer.flush()

# Call the function to produce messages
produce_messages()

print("Finished producing messages.")

```

CSV file is already placed in homedir. Now, I am going to run python code to read the first 20 rows of the CSV file and send each row twice to the credit_card_topic, ensuring there are duplicate messages for testing deduplication.

```
[student@fundos ~]$ python credit_producer_test.py 
Message delivered to credit_card_topic [0] at offset 0
Message delivered to credit_card_topic [0] at offset 1
Message delivered to credit_card_topic [0] at offset 2
Message delivered to credit_card_topic [0] at offset 3
...output omitted...
Message delivered to credit_card_topic [0] at offset 38
Message delivered to credit_card_topic [0] at offset 39
Finished producing messages.
```

We can validate with ```kafka-console-consumer``` command:

```
kafka-console-consumer --bootstrap-server :9092 --topic credit_card_topic --from-beginning --property print.key=true
5008804	{"CODE_GENDER": "M", "FLAG_OWN_CAR": "Y", "FLAG_OWN_REALTY": "Y", "CNT_CHILDREN": "0", "AMT_INCOME_TOTAL": "427500.0", "NAME_INCOME_TYPE": "Working", "NAME_EDUCATION_TYPE": "Higher education", "NAME_FAMILY_STATUS": "Civil marriage", "NAME_HOUSING_TYPE": "Rented apartment", "DAYS_BIRTH": "-12005", "DAYS_EMPLOYED": "-4542", "FLAG_MOBIL": "1", "FLAG_WORK_PHONE": "1", "FLAG_PHONE": "0", "FLAG_EMAIL": "0", "OCCUPATION_TYPE": "", "CNT_FAM_MEMBERS": "2.0"}
5008804	{"CODE_GENDER": "M", "FLAG_OWN_CAR": "Y", "FLAG_OWN_REALTY": "Y", "CNT_CHILDREN": "0", "AMT_INCOME_TOTAL": "427500.0", "NAME_INCOME_TYPE": "Working", "NAME_EDUCATION_TYPE": "Higher education", "NAME_FAMILY_STATUS": "Civil marriage", "NAME_HOUSING_TYPE": "Rented apartment", "DAYS_BIRTH": "-12005", "DAYS_EMPLOYED": "-4542", "FLAG_MOBIL": "1", "FLAG_WORK_PHONE": "1", "FLAG_PHONE": "0", "FLAG_EMAIL": "0", "OCCUPATION_TYPE": "", "CNT_FAM_MEMBERS": "2.0"}
5008805	{"CODE_GENDER": "M", "FLAG_OWN_CAR": "Y", "FLAG_OWN_REALTY": "Y", "CNT_CHILDREN": "0", "AMT_INCOME_TOTAL": "427500.0", "NAME_INCOME_TYPE": "Working", "NAME_EDUCATION_TYPE": "Higher education", "NAME_FAMILY_STATUS": "Civil marriage", "NAME_HOUSING_TYPE": "Rented apartment", "DAYS_BIRTH": "-12005", "DAYS_EMPLOYED": "-4542", "FLAG_MOBIL": "1", "FLAG_WORK_PHONE": "1", "FLAG_PHONE": "0", "FLAG_EMAIL": "0", "OCCUPATION_TYPE": "", "CNT_FAM_MEMBERS": "2.0"}
5008805	{"CODE_GENDER": "M", "FLAG_OWN_CAR": "Y", "FLAG_OWN_REALTY": "Y", "CNT_CHILDREN": "0", "AMT_INCOME_TOTAL": "427500.0", "NAME_INCOME_TYPE": "Working", "NAME_EDUCATION_TYPE": "Higher education", "NAME_FAMILY_STATUS": "Civil marriage", "NAME_HOUSING_TYPE": "Rented apartment", "DAYS_BIRTH": "-12005", "DAYS_EMPLOYED": "-4542", "FLAG_MOBIL": "1", "FLAG_WORK_PHONE": "1", "FLAG_PHONE": "0", "FLAG_EMAIL": "0", "OCCUPATION_TYPE": "", "CNT_FAM_MEMBERS": "2.0"}
...output omitted...
5008825	{"CODE_GENDER": "F", "FLAG_OWN_CAR": "Y", "FLAG_OWN_REALTY": "N", "CNT_CHILDREN": "0", "AMT_INCOME_TOTAL": "130500.0", "NAME_INCOME_TYPE": "Working", "NAME_EDUCATION_TYPE": "Incomplete higher", "NAME_FAMILY_STATUS": "Married", "NAME_HOUSING_TYPE": "House / apartment", "DAYS_BIRTH": "-10669", "DAYS_EMPLOYED": "-1103", "FLAG_MOBIL": "1", "FLAG_WORK_PHONE": "0", "FLAG_PHONE": "0", "FLAG_EMAIL": "0", "OCCUPATION_TYPE": "Accountants", "CNT_FAM_MEMBERS": "2.0"}
5008825	{"CODE_GENDER": "F", "FLAG_OWN_CAR": "Y", "FLAG_OWN_REALTY": "N", "CNT_CHILDREN": "0", "AMT_INCOME_TOTAL": "130500.0", "NAME_INCOME_TYPE": "Working", "NAME_EDUCATION_TYPE": "Incomplete higher", "NAME_FAMILY_STATUS": "Married", "NAME_HOUSING_TYPE": "House / apartment", "DAYS_BIRTH": "-10669", "DAYS_EMPLOYED": "-1103", "FLAG_MOBIL": "1", "FLAG_WORK_PHONE": "0", "FLAG_PHONE": "0", "FLAG_EMAIL": "0", "OCCUPATION_TYPE": "Accountants", "CNT_FAM_MEMBERS": "2.0"}
^CProcessed a total of 40 messages
```

2. Now that we have produced some messages with duplicates, we will proceed to deduplicate the messages.

As per recommendations, I tried to figure out deduplication mechanism with help of ksqlDB. 

I used following command to access the ksqlDB CLI to interact with Kafka topic:

```
# docker exec -it ksqldb-cli ksql http://ksqldb-server:8088
                  
                  ===========================================
                  =       _              _ ____  ____       =
                  =      | | _____  __ _| |  _ \| __ )      =
                  =      | |/ / __|/ _` | | | | |  _ \      =
                  =      |   <\__ \ (_| | | |_| | |_) |     =
                  =      |_|\_\___/\__, |_|____/|____/      =
                  =                   |_|                   =
                  =        The Database purpose-built       =
                  =        for stream processing apps       =
                  ===========================================

Copyright 2017-2022 Confluent Inc.

CLI v7.8.0, Server v7.8.0 located at http://ksqldb-server:8088
Server Status: RUNNING
```

Next, I've created an input stream based on the _credit_card_topic_:

```
ksql> CREATE STREAM cc_stream (
>  ID VARCHAR KEY,
>  CODE_GENDER VARCHAR,
>  FLAG_OWN_CAR VARCHAR,
>  FLAG_OWN_REALTY VARCHAR,
>  CNT_CHILDREN VARCHAR,
>  AMT_INCOME_TOTAL VARCHAR,
>  NAME_INCOME_TYPE VARCHAR,
>  NAME_EDUCATION_TYPE VARCHAR,
>  NAME_FAMILY_STATUS VARCHAR,
>  NAME_HOUSING_TYPE VARCHAR,
>  DAYS_BIRTH VARCHAR,
>  DAYS_EMPLOYED VARCHAR,
>  FLAG_MOBIL VARCHAR,
>  FLAG_WORK_PHONE VARCHAR,
>  FLAG_PHONE VARCHAR,
>  FLAG_EMAIL VARCHAR,
>  OCCUPATION_TYPE VARCHAR,
>  CNT_FAM_MEMBERS VARCHAR
>) WITH (
>  KAFKA_TOPIC='credit_card_topic',
>  VALUE_FORMAT='JSON',
>  PARTITIONS=1
>);

 Message        
----------------
 Stream created 
----------------
```

Followed by deduplication/filtering table:

```
ksql> CREATE TABLE dedup_cc_table AS
>SELECT ID,
>       LATEST_BY_OFFSET(CODE_GENDER) AS CODE_GENDER,
>       LATEST_BY_OFFSET(FLAG_OWN_CAR) AS FLAG_OWN_CAR,
>       LATEST_BY_OFFSET(FLAG_OWN_REALTY) AS FLAG_OWN_REALTY,
>       LATEST_BY_OFFSET(CNT_CHILDREN) AS CNT_CHILDREN,
>       LATEST_BY_OFFSET(AMT_INCOME_TOTAL) AS AMT_INCOME_TOTAL,
>       LATEST_BY_OFFSET(NAME_INCOME_TYPE) AS NAME_INCOME_TYPE,
>       LATEST_BY_OFFSET(NAME_EDUCATION_TYPE) AS NAME_EDUCATION_TYPE,
>       LATEST_BY_OFFSET(NAME_FAMILY_STATUS) AS NAME_FAMILY_STATUS,
>       LATEST_BY_OFFSET(NAME_HOUSING_TYPE) AS NAME_HOUSING_TYPE,
>       LATEST_BY_OFFSET(DAYS_BIRTH) AS DAYS_BIRTH,
>       LATEST_BY_OFFSET(DAYS_EMPLOYED) AS DAYS_EMPLOYED,
>       LATEST_BY_OFFSET(FLAG_MOBIL) AS FLAG_MOBIL,
>       LATEST_BY_OFFSET(FLAG_WORK_PHONE) AS FLAG_WORK_PHONE,
>       LATEST_BY_OFFSET(FLAG_PHONE) AS FLAG_PHONE,
>       LATEST_BY_OFFSET(FLAG_EMAIL) AS FLAG_EMAIL,
>       LATEST_BY_OFFSET(OCCUPATION_TYPE) AS OCCUPATION_TYPE,
>       LATEST_BY_OFFSET(CNT_FAM_MEMBERS) AS CNT_FAM_MEMBERS
>FROM cc_stream
>GROUP BY ID;

 Message                                      
----------------------------------------------
 Created query with ID CTAS_DEDUP_CC_TABLE_89 
----------------------------------------------
ksql> show tables;

 Table Name     | Kafka Topic    | Key Format | Value Format | Windowed 
------------------------------------------------------------------------
 DEDUP_CC_TABLE | DEDUP_CC_TABLE | KAFKA      | JSON         | false    
------------------------------------------------------------------------
ksql> 
```

New topic associated to the _dedup_cc_table_ was also created. However, when consuming messages from the this topic, results did not meet task criteria - it still had duplicated messages.

```
kafka-console-consumer --bootstrap-server :9092 --topic DEDUP_CC_TABLE --from-beginning --property print.key=true
5008804	{"CODE_GENDER":"M","FLAG_OWN_CAR":"Y","FLAG_OWN_REALTY":"Y","CNT_CHILDREN":"0","AMT_INCOME_TOTAL":"427500.0","NAME_INCOME_TYPE":"Working","NAME_EDUCATION_TYPE":"Higher education","NAME_FAMILY_STATUS":"Civil marriage","NAME_HOUSING_TYPE":"Rented apartment","DAYS_BIRTH":"-12005","DAYS_EMPLOYED":"-4542","FLAG_MOBIL":"1","FLAG_WORK_PHONE":"1","FLAG_PHONE":"0","FLAG_EMAIL":"0","OCCUPATION_TYPE":"","CNT_FAM_MEMBERS":"2.0"}
5008804	{"CODE_GENDER":"M","FLAG_OWN_CAR":"Y","FLAG_OWN_REALTY":"Y","CNT_CHILDREN":"0","AMT_INCOME_TOTAL":"427500.0","NAME_INCOME_TYPE":"Working","NAME_EDUCATION_TYPE":"Higher education","NAME_FAMILY_STATUS":"Civil marriage","NAME_HOUSING_TYPE":"Rented apartment","DAYS_BIRTH":"-12005","DAYS_EMPLOYED":"-4542","FLAG_MOBIL":"1","FLAG_WORK_PHONE":"1","FLAG_PHONE":"0","FLAG_EMAIL":"0","OCCUPATION_TYPE":"","CNT_FAM_MEMBERS":"2.0"}
...output omitted...
^CProcessed a total of 40 messages
```

When I made a query to the table - we've got expected result:

```
ksql> select * FROM DEDUP_CC_TABLE;
+------+------+------+------+------+------+------+------+------+------+------+------+------+------+------+------+------+------+
|ID    |CODE_G|FLAG_O|FLAG_O|CNT_CH|AMT_IN|NAME_I|NAME_E|NAME_F|NAME_H|DAYS_B|DAYS_E|FLAG_M|FLAG_W|FLAG_P|FLAG_E|OCCUPA|CNT_FA|
|      |ENDER |WN_CAR|WN_REA|ILDREN|COME_T|NCOME_|DUCATI|AMILY_|OUSING|IRTH  |MPLOYE|OBIL  |ORK_PH|HONE  |MAIL  |TION_T|M_MEMB|
|      |      |      |LTY   |      |OTAL  |TYPE  |ON_TYP|STATUS|_TYPE |      |D     |      |ONE   |      |      |YPE   |ERS   |
|      |      |      |      |      |      |      |E     |      |      |      |      |      |      |      |      |      |      |
+------+------+------+------+------+------+------+------+------+------+------+------+------+------+------+------+------+------+
|500880|M     |Y     |Y     |0     |427500|Workin|Higher|Civil |Rented|-12005|-4542 |1     |1     |0     |0     |      |2.0   |
|4     |      |      |      |      |.0    |g     | educa|marria| apart|      |      |      |      |      |      |      |      |
|      |      |      |      |      |      |      |tion  |ge    |ment  |      |      |      |      |      |      |      |      |
|500880|M     |Y     |Y     |0     |427500|Workin|Higher|Civil |Rented|-12005|-4542 |1     |1     |0     |0     |      |2.0   |
|5     |      |      |      |      |.0    |g     | educa|marria| apart|      |      |      |      |      |      |      |      |
|      |      |      |      |      |      |      |tion  |ge    |ment  |      |      |      |      |      |      |      |      |
|500880|M     |Y     |Y     |0     |112500|Workin|Second|Marrie|House |-21474|-1134 |1     |0     |0     |0     |Securi|2.0   |
|6     |      |      |      |      |.0    |g     |ary / |d     |/ apar|      |      |      |      |      |      |ty sta|      |
|      |      |      |      |      |      |      |second|      |tment |      |      |      |      |      |      |ff    |      |
|      |      |      |      |      |      |      |ary sp|      |      |      |      |      |      |      |      |      |      |
|      |      |      |      |      |      |      |ecial |      |      |      |      |      |      |      |      |      |      |
|500880|F     |N     |Y     |0     |270000|Commer|Second|Single|House |-19110|-3051 |1     |0     |1     |1     |Sales |1.0   |
|8     |      |      |      |      |.0    |cial a|ary / | / not|/ apar|      |      |      |      |      |      |staff |      |
|      |      |      |      |      |      |ssocia|second| marri|tment |      |      |      |      |      |      |      |      |
|      |      |      |      |      |      |te    |ary sp|ed    |      |      |      |      |      |      |      |      |      |
|      |      |      |      |      |      |      |ecial |      |      |      |      |      |      |      |      |      |      |
...output omitted...
|511295|M     |Y     |Y     |0     |270000|Workin|Higher|Marrie|House |-16872|-769  |1     |1     |1     |1     |Accoun|2.0   |
|6     |      |      |      |      |.0    |g     | educa|d     |/ apar|      |      |      |      |      |      |tants |      |
|      |      |      |      |      |      |      |tion  |      |tment |      |      |      |      |      |      |      |      |
|615365|M     |Y     |Y     |0     |270000|Workin|Higher|Marrie|House |-16872|-769  |1     |1     |1     |1     |Accoun|2.0   |
|1     |      |      |      |      |.0    |g     | educa|d     |/ apar|      |      |      |      |      |      |tants |      |
|      |      |      |      |      |      |      |tion  |      |tment |      |      |      |      |      |      |      |      |
Query terminated
```

In the end, after trying-testing multiple options with streams/tables, I could not figure out correct method to produce deduplicated data into new topic yet.

