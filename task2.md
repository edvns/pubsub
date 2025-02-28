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
        print("Message delivery failed: {err}")
    else:
        print("Message delivered to {msg.topic()} [{msg.partition()}] at offset {msg.offset()}")

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

```


2. 


