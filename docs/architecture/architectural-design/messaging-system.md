---
title: 3.3 Messaging System
doc_status: unreviewed
last_reviewed:
reviewed_by:
review_notes:
---
### 3.3 Messaging System

Messaging pervades system structure, allowing components to collaborate with loose coupling. It provides scalability and higher availability, for example by allowing multiple instances of services to operate independently by processing the same messages. It allows greater extensibility and replaceability of components. It makes development of components more independent so that multiple sub-teams can contribute to different areas simultaneously and release at different times. 

The messaging system can also be used to stream events, such as billing-relevant events being generated across the system and recorded by the accounting component. 

Apache Pulsar is used for the messaging system, which, via its distributed ledger, can serve the dual role of streaming events and reliable distributed messaging. Pulsar can also perform indefinite retention, such as of billing-related events. Comparing its main competitors, Kafka is an effective tool for event streaming but is less able to handle messaging in which individual messages are processed and acknowledged in a non-linear order. Conversely, RabbitMQ is designed for message-passing and has rich features for routing and queuing which we don’t need but is not designed for streaming events or storing messages for longer periods. 

