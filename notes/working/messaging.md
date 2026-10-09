### Amazon AppFlow

Managed integration service for data transfer between source and target data sources. Works for over 80 cloud services.  
Flow triggers:

- Run on demand \- users manually run the flow as needed
- Run on event \- AppFlow runs the flow in response to an event from an SaaS application
- Run on schedule \- AppFlow runs the flow on a recurring schedule

Features:

- Create dataflows between applications within minutes
- Aggregate data from multiple sources
- Data encryption possible at rest or in transit
- Partition and aggregation settings available to optimise query performance
- Develop custom connectors via the AppFlow custom connector SDKs
- Create private flows via PrivateLink
- Catalogue data transferred to S3 via AWS Glue Data Catalog

## Simple Notification Service (SNS)

Highly available, durable, secure and fully-managed pub/sub messaging service enabling the decoupling of microservices, distributed systems and serverless applications. Event bus is called the SNS Topic.  
Sources (publications):

- Tonnes of AWS service options
- Publish to Amazon SNS topics standard
- Most publish FIFO

Destinations (subscribers):

- Application-to-application (A2A)
  - Data Firehose
  - Lambda Functions
  - SQS Queue
  - HTTP/S endpoint
  - AWS Event Fork Pipelines
- Application-to-person (A2P)
  - Mobile apps
  - Mobile phone numbers
  - Email address
  - AWS Chatbot
  - PagerDuty

Topics:

- Group subscriptions together
- Automatically format messages when delivering to subscribers
  - Can deliver to multiple protocols at once
  - Publishers don’t have to care about subscriber protocols
- Can be encrypted via KMS
- Standard
  - High throughput
  - Delivered at least once (duplicates possible)
  - No guaranteed order
  - Useful for high volume messages without ordering or delivery being crucial (e.g. alerts/notifications)
- FIFO (guarantees order)
  - FIFO topics must have .fifo on end of topic name and attribute FifoTopic=true
  - Lower throughput than standard
  - Delivered exactly once (no duplicates)
  - Messages delivered in exact order they get sent to the message group
  - Useful when order and exact delivery are important (e.g. bank transactions/ordered data processing)
  - Supports message grouping \- allows multiple ordered streams within the same topic

### Messages

When publishing messages \>256KB, need to use SNS Extended Client Library (which goes up to 2GB). Extended client uses S3.  
Message Attributes can be delivered too, which provide structured metadata items about the message (String, String.Array, Number, Binary data types supported).  
Batch processing possible up to 10 batch size \- reduces costs.

### Subscriptions

A subscription is needed to receive messages from a topic. A subscription can only subscribe to 1 protocol and 1 topic.  
Protocols:

- HTTP/S \- create webhooks for web application
- Email \- good for internal notifications (plain text only)
- Email-JSON
- Amazon SQS \- place message into queue
- AWS Lambda \- trigger some process
- SMS
- Platform application endpoint \- mobile push

### Filter Policy

Allows filtering of messages to deliver based on either the message attributes, or message body.  
Options:

- AND logic
- OR logic
- OR operator (not sure of the difference from above)
- Key matching
- Numeric value
  - Exact matching
  - Anything but (not) matching
  - Range matching
- String value
  - Exact matching
  - Anything but matching
  - Match using prefix with anything but operator
  - Case insensitive matching
  - IP address matching
  - Prefix matching
  - Suffix matching

### Message Data Protection

Safeguards data published to SNS topics by using protection policies to audit, mask, redact or block sensitive information moving between applications or services.  
Scans for:

- Personally Identifiable Information (PII)
- Protected Health Information (PHI)
- Predefined data identifiers
  - Names
  - Addresses
  - Credit card numbers
- Custom data identifiers

Actions:

- Audit \- up to 99% of data published (for some reason)
  - Send findings to CloudWatch, S3 or Data Firehose
  - De-identify \- mask / redact data
  - Deny \- block data being sent

Useful for reducing risk and regulatory compliance.  
Only supported for standard SNS topics.

### Raw Message Delivery

Avoid having Amazon Data Firehose, Amazon SQS or HTTP/S endpoints process the JSON formatting of messages. Use if not wanting to parse on the other side or you need minimised payload size.  
Data Firehose / SQS \- metadata stripped and message sent as-is  
HTTP/S endpoint \- raw delivery header set to true

### Delivery Policy

Defines how SNS retries message delivery when server-side errors occur. Each protocol has its own. Cannot be changed (except HTTP/S endpoints).  
Application to application (A2A) (Data Firehose, Lambda, SQS):

- 3 immediate retries \- no delay
- 2 retries \- 1s delay
- 10 retries with exponential backoff \- 1s-20s delay
- 100,000 retries \- 20s delay

Application to person (A2P) (SMTP, SNS, Mobile Push):

- 2 retries \- 10s delay
- 10 retries with exponential backoff \- 10s-600s (10mins)
- 38 retries \- 600s (10m) delay

HTTP/S endpoints can be custom. Backoff types:

- Arithmetic
- Exponential
- Geometric
- Linear

### Dead Letter Queue

Will send failed message attempts to an SQS queue. Topic and Queue type must match (standard-\>standard, FIFO-\>FIFO). SNS topic configured to send to a DLQ via the CLI. Queue policy configuration needed to allow SNS to send events to the SQS queue.

### Application As Subscriber

Push notification messages can be sent directly to apps on mobile devices. Protocol type is ‘Platform application endpoint’.

### Simple Queuing Service (SQS)

Provides asynchronous communication across decoupled senders and receivers (producers / consumers). Pulling is required \- not real-time and not reactive.  
Types:

- Standard (default)
  - Almost unlimited messages
  - Order not guaranteed (but generally in order)
  - Delivered at least once (duplicates possible)
  - Can batch similar to SNS
- FIFO
  - Limited messages (300 transactions per second)
  - Order guaranteed (using unique message group id)
  - No duplicates guaranteed
  - Polling required unique id
  - Can read up to 10 messages at once
  - Managed in partitions across multiple AZs by AWS
  - Cannot convert from standard
  - Can batch in 10 for more throughput

Message size \= 1 byte \-\> 256KB. Amazon SQS Extended Client Library required for larger messages. Max then of 2GB. Saves payload to S3 bucket and references.  
Message retention \= 60s \-\> 14 days (default is 4 days. After this, the message is dropped from the queue (deleted).  
Queue can be encrypted using Amazon-managed server-side-encryption (SSE) or using KMS.  
Message metadata can be attached similar to SNS in following types:

- String
- Number
- Binary
- Custom
  - Append custom type label to any data type (e.g. Number.byte, Number.short, Binary.gif, Binary.png)

### Attribute-Based Access Control (ABAC)

Authorisation process defining permissions based on tags attached to users and AWS resources. SQS supports ABAC by allowing access control to SQS queues based on tags and aliases associated with it.  
Condition tags:

- aws:ResourceTag
- aws:RequestTag
- aws:TagKeys

Example: denying production resources (tagged with prod) from sending, receiving or deleting messages in a queue (aws:ResourceTag/environment: prod).  
Access Policies are also possible \- like IAM policies, granting other principals permissions to the SQS queue.

### Visibility Timeout

Period of time a message will be invisible after being read/consumed by an application to avoid being processed by other applications. A message is hidden only after it is consumed from the queue. Set VisibilityTimeout via the CLI.  
Default \= 30s, Min \= 0s, Max \= 43200s (12h).

### Delay Queues

Postpone delivery of new messages to consumers for a specified time period when your app needs more time. Any messages sent to a delay queue remain invisible to consumers for the duration of this period. A message is hidden when first added to the queue. Set DelaySeconds via the CLI.  
Default \= 0s, Max \= 900s (15m)  
Standard queues apply the delay setting to new messages in the queue, FIFO queues apply the delay setting to all messages in the queue.

### Message Timers

Allow specifying an initial invisibility period for an individual message when sending to a queue. Uses same DelaySeconds property as delay queues but this is at message level. Not supported by FIFO queues.

### Temporary Queues

High-throughput, cost-effective, application managed temporary queues when using common message patterns like request-response. Temporary Queue Client allows the creation of lightweight queues automatically deleted when no longer in use.  
Benefits:

- Lightweight communication channels for specific threads or processes
- Can be created / deleted without additional cost
- API compatible with standard SQS queues

### Short vs. Long Polling

Short returns messages immediately (even if empty). Long waits until a message arrives in the queue or until the poll timeout. Short is the default. Short good if message required right away, long better for saving polling costs.

### Amazon MQ

Managed message broker services for Apache ActiveMQ and RabbitMQ. Similar to SQS but MQ can handle more complex delivery rules with different performance guarantees. Rabbit MQ is more lightweight and performant, ideal for complex routing or high throughput requirements. ActiveMQ is older and generally performant but extra features can add overhead. Both provide support for AMQP, MQTT and STOMP protocols \- ActiveMQ also supports OpenWire.

#### Advanced Message Queuing Protocol (AMQP)

Open standard wire-level protocol designed for messaging middleware enabling confirming client applications to communicate with conforming messaging middleware servers.

- Publishers publish messages to exchanges
- Exchanges distribute message copies to queues using rules called bindings
- Broker pushes messages to subscribed consumers OR consumers pull messages from queues
- Messages can have metadata attached
- Messages are only removed from the queue when a consumer ACKs (acknowledges) the broker

Exchange types:

- Direct (default)
- Fanout
- Topic
- Headers

#### Message Queue Telemetry Transport (MQTT)

Lightweight pub/sub messaging protocol using minimal bandwidth. Common use cases: IoT, real-time messaging apps. Suitable for machine-to-machine (M2M) communication.

- Publishers publish to MQTT brokers
- Subscribers pull or brokers push
- Use MQTT client to publish or subscribe programmatically
- Publish to or subscribe on topic names

#### Simple Text-Oriented Messaging Protocol (STOMP)

Simple, text-based wire protocol allowing clients to communicate with almost any message broker. Very simple. Can be used with telnet clients.

## Amazon MSK

Amazon Managed Streaming for Apache Kafka (MSK) is a fully-managed service enabling the building and running of applications that use Apache Kafka to process streaming data.

- Uses Zookeeper servers
  - Does not support KRaft
- Types of nodes:
  - Broker nodes \- handle storage and processing of messages
  - Zookeeper nodes (Znodes) \- manage overall structure of the cluster
- Cluster types:
  - Provisioned \- manage broker instances manually
  - Serverless \- automatic instance management, pay for usage
- Has direct integration with:
  - S3
  - EventBridge Pipes

Launched within your VPC and connections must originate from the same VPC. Public access can be enabled on clusters after their launch.

**Bootstrap brokers** are broker endpoints that a Kafka client can use as a starting point to connect to the cluster. BootstrapBrokerStringPublicSasllam \= public access, BootstrapBrokerStringSasllam for access from within AWS. The number of bootstrap broker endpoints will depend on how many AZs the cluster is deployed in.

**ZooKeeper connection string URLs** are used with Kafka to specify the host and port of the ZooKeeper ensemble that Kafka should connect to for managing cluster metadata and coordination such as:

- Broker registration
- Topic configuration
- Cluster membership
- Quota management
- Access Control Lists (ACLs)

Describe-cluster can be used to get the connection url (apart from when using serverless clusters).

**MSK Connect** is a feature allowing developers to easily stream data to and from Kafka clusters. Uses Kafka Connect open-source framework for connecting clusters with external systems like databases, search indexes and file systems.

1. Download (or create) Kafka Connect plugins
2. Upload to S3
3. Create a plugin in MSK Connect.
4. Create a connector specifying plugin and config info to the source

**Apache Kafka** is an open-source streaming platform used to create high-performance data pipelines, streaming analytics, data integration and mission-critical applications.

- Originally built by LinkedIn
- Written in Scala and Java
- Data stored in partitions on a Kafka Cluster
  - Partitions can span multiple machines (distributed computing)
- Producers publish messages in a key/value format using Kafka Producer API
- Consumers listen for messages and consume using the Kafka Consumer API
- Messages organised into Topics
  - Producers push onto topics and consumers listen to them
- Can interact with Kafka using Kafka CLI scripts
- Can use a programming SDK in various languages

**Apache ZooKeeper** is an open-source server for highly-reliable distributed coordination of cloud applications. It exposes common services into a simple interface so you don’t have to write them from scratch \- such as:

- Naming
- Config management
- Synchronisation
- Group services

Used by:

- Apache Hadoop
- Apache Kafka
- Apache Solr
- Apache Hbase
- Apache Accumulo
- Apache Druid
- Apache Helix
