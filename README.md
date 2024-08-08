# Introduction: IVCAP Queue Service
The IVCAP Queue endpoint (`/1/queues`) provides APIs for managing queues and performing queue operations like enqueuing and dequeuing messages. Queues are a fundamental component of an event-driven architecture, enabling decoupled communication between producers and consumers. With this endpoint, you can create new queues, list existing ones, retrieve queue details, delete queues, as well as send messages to a queue (enqueue) and retrieve messages from a queue (dequeue). The Queue endpoint supports to give you fine-grained control over queue management.

# Ocean Temperature Data Processing System

This project demonstrates a distributed system for processing environmental data, specifically ocean temperature measurements collected from various buoys. This simple example showcases a practical application of using the IVCAP Queue endpoint in distributed systems in the scientific domain.

## Overview

The system consists of two main components:

1. **Producer**: Simulates the collection of temperature data from buoys and publishes this data to the IVCAP Queue.
2. **Consumer**: Consumes the data from the IVCAP Queue, processes it, and logs the data.

## Components

### Producer

The producer script (`producer.py`) simulates the collection of temperature data from buoys. It generates random temperature readings and publishes them to a specified IVCAP Queue.

The producer accepts the following parameters:

- `number_of_data_points`: The number of data points (temperature readings) to publish to the queue.
- `queue_id`: The identifier of the IVCAP Queue to publish the data to.

### Consumer

The consumer script (`consumer.py`) retrieves and logs the temperature data from the specified IVCAP Queue.

The consumer accepts the following parameters:

- `subscription_duration`: The duration in minutes to consume data from the queue.
- `queue_id`: The identifier of the IVCAP Queue to consume data from.

The consumer continuously checks the queue for new data and logs any received temperature measurements until the specified subscription duration has elapsed.

## Getting Started

1. Set up your IVCAP environment and authenticate with the IVCAP CLI.
2. Create an IVCAP Queue using the IVCAP CLI or the IVCAP Console.
    ```
    ❯ ivcap queue create -d "example queue" ocean
    Queue urn:ivcap:queue:5e245a11-04f9-593e-b456-3e831474381e created
    ```
3. Build and register the producer and consumer services with IVCAP using the `service-register` Make target.
    ```
    cd producer
    make service-register

    cd ../consumer
    make service-register
    ```
4. To view the services just created:
    ```
    ❯ ivcap service list
    +----+------------------------+--------------------------------------------------------+
    | ID | NAME                   | ACCOUNT                                                |
    +----+------------------------+--------------------------------------------------------+
    | @1 | Queue Consumer         | urn:ivcap:account:e91bb525-83b7-4641-9fb8-b61d084a9b81 |
    | @2 | Queue Producer         | urn:ivcap:account:e91bb525-83b7-4641-9fb8-b61d084a9b81 |
    +----+------------------------+--------------------------------------------------------+

    ❯ ivcap service get @1

            ID  urn:ivcap:service:e3b33fc9-84be-5bc9-8363-28de684d64b9:queue-consumer-service (@1)
            Name  Queue Consumer
    Description  A service that consumes synthetic buoy data from a queue.
        Status  active
    Account ID  urn:ivcap:account:e91bb525-83b7-4641-9fb8-b61d084a9b81
    Parameters  ┌───────────────────────┬────────────────────────────────┬────────┬─────────┬──────────┐
                │ NAME                  │ DESCRIPTION                    │ TYPE   │ DEFAULT │ OPTIONAL │
                ├───────────────────────┼────────────────────────────────┼────────┼─────────┼──────────┤
                │ subscription_duration │ The duration in minutes to con │ int    │         │ false    │
                │                       │ sume data from the queue.      │        │         │          │
                ├───────────────────────┼────────────────────────────────┼────────┼─────────┼──────────┤
                │              queue_id │ The identifier of the queue to │ string │         │ false    │
                │                       │  consume data from.            │        │         │          │
                └───────────────────────┴────────────────────────────────┴────────┴─────────┴──────────┘

    ❯ ivcap service get @2

            ID  urn:ivcap:service:e5fb1c5f-79ed-55bc-ab86-89aa44c8055c:queue-producer-service (@1)
            Name  Queue Producer
    Description  A service that publishes synthetic buoy data to a queue.
        Status  active
    Account ID  urn:ivcap:account:e91bb525-83b7-4641-9fb8-b61d084a9b81
    Parameters  ┌───────────────────────┬────────────────────────────────┬────────┬─────────┬──────────┐
                │ NAME                  │ DESCRIPTION                    │ TYPE   │ DEFAULT │ OPTIONAL │
                ├───────────────────────┼────────────────────────────────┼────────┼─────────┼──────────┤
                │ number_of_data_points │ Number of data points to publi │ int    │         │ false    │
                │                       │ sh to the queue.               │        │         │          │
                ├───────────────────────┼────────────────────────────────┼────────┼─────────┼──────────┤
                │              queue_id │ The identifier of the queue to │ string │         │ false    │
                │                       │  publish data to.              │        │         │          │
                └───────────────────────┴────────────────────────────────┴────────┴─────────┴──────────┘
    ```
5. Run the producer service with the appropriate parameters to publish synthetic temperature data to the queue.
    ```
    ❯ ivcap orders create --name "queue-producer" urn:ivcap:service:e5fb1c5f-79ed-55bc-ab86-89aa44c8055c:queue-producer-service number_of_data_points=5 queue_id=urn:ivcap:queue:5e245a11-04f9-593e-b456-3e831474381e
    Order 'urn:ivcap:order:8f23007c-1b72-492e-aa29-c3ff10e3edd9' with status 'pending' submitted.

    ❯ ivcap orders logs urn:ivcap:order:8f23007c-1b72-492e-aa29-c3ff10e3edd9
    INFO 2024-08-08T01:38:00+0000 ivcap IVCAP Service 'Queue Producer' /7cf40e8 (sdk 0.8.0) built on Thu  8 Aug 2024 11:02:12 AEST.
    INFO 2024-08-08T01:38:00+0000 ivcap Checking for data-proxy at 'http://artifact.local/readyz'.
    INFO 2024-08-08T01:38:00+0000 ivcap Data-proxy doesn't seem to be ready yet, will wait 10sec and try again.
    INFO 2024-08-08T01:38:10+0000 ivcap Checking for data-proxy at 'http://artifact.local/readyz'.
    INFO 2024-08-08T01:38:10+0000 ivcap Starting order 'urn:ivcap:order:ed88590b-047b-4ecc-88a6-2b54a9175e09' for service 'Queue Producer' on node 'order-ed88590b-047b-4ecc-88a6-2b54a9175e09'
    INFO 2024-08-08T01:38:10+0000 ivcap Starting service with 'ServiceArgs(number_of_data_points=5, queue_id='urn:ivcap:queue:5e245a11-04f9-593e-b456-3e831474381e')'
    DEBUG 2024-08-08T01:38:10+0000 ivcap Read queue with ID: urn:ivcap:queue:5e245a11-04f9-593e-b456-3e831474381e
    INFO 2024-08-08T01:38:10+0000 service Retrieved queue 'ocean' with ID urn:ivcap:queue:5e245a11-04f9-593e-b456-3e831474381e.
    INFO 2024-08-08T01:38:10+0000 service Published data: {'temperature': 18.725118563657446, 'location': 'Buoy106', 'timestamp': '2024-08-08T01:38:10Z'} and received response: {'id': 'urn:ivcap:message:5e245a11-04f9-593e-b456-3e831474381e:1'}
    INFO 2024-08-08T01:38:11+0000 service Published data: {'temperature': 12.61173874572024, 'location': 'Buoy140', 'timestamp': '2024-08-08T01:38:11Z'} and received response: {'id': 'urn:ivcap:message:5e245a11-04f9-593e-b456-3e831474381e:2'}
    INFO 2024-08-08T01:38:11+0000 service Published data: {'temperature': 21.466007423271925, 'location': 'Buoy104', 'timestamp': '2024-08-08T01:38:11Z'} and received response: {'id': 'urn:ivcap:message:5e245a11-04f9-593e-b456-3e831474381e:3'}
    INFO 2024-08-08T01:38:11+0000 service Published data: {'temperature': 18.901966185830716, 'location': 'Buoy131', 'timestamp': '2024-08-08T01:38:11Z'} and received response: {'id': 'urn:ivcap:message:5e245a11-04f9-593e-b456-3e831474381e:4'}
    INFO 2024-08-08T01:38:11+0000 service Published data: {'temperature': 24.49704226953446, 'location': 'Buoy189', 'timestamp': '2024-08-08T01:38:11Z'} and received response: {'id': 'urn:ivcap:message:5e245a11-04f9-593e-b456-3e831474381e:5'}
    INFO 2024-08-08T01:38:11+0000 service Added 5 tasks to the queue.
    ```
6. Run the consumer service with the appropriate parameters to consume and log the temperature data from the queue.
    ```
    ❯ ivcap orders create --name "queue-consume" urn:ivcap:service:e3b33fc9-84be-5bc9-8363-28de684d64b9:queue-consumer-service subscription_duration=1 queue_id=urn:ivcap:queue:5e245a11-04f9-593e-b456-3e831474381e
    Order 'urn:ivcap:order:6ec17d69-7aeb-485d-968c-3cad86a58559' with status 'pending' submitted.

    ❯ ivcap orders logs urn:ivcap:order:6ec17d69-7aeb-485d-968c-3cad86a58559
    INFO 2024-08-08T01:38:53+0000 ivcap IVCAP Service 'Queue Consumer' /7cf40e8 (sdk 0.8.0) built on Thu  8 Aug 2024 11:25:16 AEST.
    INFO 2024-08-08T01:38:53+0000 ivcap Checking for data-proxy at 'http://artifact.local/readyz'.
    INFO 2024-08-08T01:38:53+0000 ivcap Data-proxy doesn't seem to be ready yet, will wait 10sec and try again.
    INFO 2024-08-08T01:39:03+0000 ivcap Checking for data-proxy at 'http://artifact.local/readyz'.
    INFO 2024-08-08T01:39:03+0000 ivcap Starting order 'urn:ivcap:order:c6f1ecc2-7e65-4b72-8020-b9817b2b941d' for service 'Queue Consumer' on node 'order-c6f1ecc2-7e65-4b72-8020-b9817b2b941d'
    INFO 2024-08-08T01:39:03+0000 ivcap Starting service with 'ServiceArgs(subscription_duration=1, queue_id='urn:ivcap:queue:5e245a11-04f9-593e-b456-3e831474381e')'
    DEBUG 2024-08-08T01:39:03+0000 ivcap Read queue with ID: urn:ivcap:queue:5e245a11-04f9-593e-b456-3e831474381e
    INFO 2024-08-08T01:39:03+0000 service Retrieved queue ocean with ID urn:ivcap:queue:5e245a11-04f9-593e-b456-3e831474381e.
    DEBUG 2024-08-08T01:39:03+0000 ivcap Dequeue messages from queue with ID: urn:ivcap:queue:5e245a11-04f9-593e-b456-3e831474381e
    INFO 2024-08-08T01:39:03+0000 service Consumed data: {'messages': [{'id': 'urn:ivcap:message:5e245a11-04f9-593e-b456-3e831474381e:1', 'content': '{"temperature": 18.725118563657446, "location": "Buoy106", "timestamp": "2024-08-08T01:38:10Z"}', 'schema': 'urn:ivcap:schema:queue:message.1', 'content-type': 'application/json'}], 'at-time': '2024-08-08T01:39:03Z'}
    DEBUG 2024-08-08T01:39:03+0000 ivcap Dequeue messages from queue with ID: urn:ivcap:queue:5e245a11-04f9-593e-b456-3e831474381e
    INFO 2024-08-08T01:39:03+0000 service Consumed data: {'messages': [{'id': 'urn:ivcap:message:5e245a11-04f9-593e-b456-3e831474381e:2', 'content': '{"temperature": 12.61173874572024, "location": "Buoy140", "timestamp": "2024-08-08T01:38:11Z"}', 'schema': 'urn:ivcap:schema:queue:message.1', 'content-type': 'application/json'}], 'at-time': '2024-08-08T01:39:03Z'}
    DEBUG 2024-08-08T01:39:03+0000 ivcap Dequeue messages from queue with ID: urn:ivcap:queue:5e245a11-04f9-593e-b456-3e831474381e
    INFO 2024-08-08T01:39:03+0000 service Consumed data: {'messages': [{'id': 'urn:ivcap:message:5e245a11-04f9-593e-b456-3e831474381e:3', 'content': '{"temperature": 21.466007423271925, "location": "Buoy104", "timestamp": "2024-08-08T01:38:11Z"}', 'schema': 'urn:ivcap:schema:queue:message.1', 'content-type': 'application/json'}], 'at-time': '2024-08-08T01:39:03Z'}
    DEBUG 2024-08-08T01:39:03+0000 ivcap Dequeue messages from queue with ID: urn:ivcap:queue:5e245a11-04f9-593e-b456-3e831474381e
    INFO 2024-08-08T01:39:03+0000 service Consumed data: {'messages': [{'id': 'urn:ivcap:message:5e245a11-04f9-593e-b456-3e831474381e:4', 'content': '{"temperature": 18.901966185830716, "location": "Buoy131", "timestamp": "2024-08-08T01:38:11Z"}', 'schema': 'urn:ivcap:schema:queue:message.1', 'content-type': 'application/json'}], 'at-time': '2024-08-08T01:39:03Z'}
    DEBUG 2024-08-08T01:39:03+0000 ivcap Dequeue messages from queue with ID: urn:ivcap:queue:5e245a11-04f9-593e-b456-3e831474381e
    INFO 2024-08-08T01:39:03+0000 service Consumed data: {'messages': [{'id': 'urn:ivcap:message:5e245a11-04f9-593e-b456-3e831474381e:5', 'content': '{"temperature": 24.49704226953446, "location": "Buoy189", "timestamp": "2024-08-08T01:38:11Z"}', 'schema': 'urn:ivcap:schema:queue:message.1', 'content-type': 'application/json'}], 'at-time': '2024-08-08T01:39:03Z'}
    ```
