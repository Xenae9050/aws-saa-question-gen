## CloudWatch

Observability is the ability to measure and understand how internal systems work using:

- Metrics \- number representing facet of the system, measured over time
- Logs \- text file recording timestamped data about events within the system
- Traces \- history of requests travelling through multiple internal services
- Alarms (sometimes called fourth pillar) \- notifications usually triggered by metrics crossing thresholds

AWS CloudWatch is a collection of monitoring tools including:

- **Logs**
- **Metrics** \- variable monitored over time using time-ordered set of data points
  - Pre-defined metric depending on service
  - Custom metrics
    - Can be high resolution (intervals of 1s-\>1m, standard res is 1m)
- **Events** \- now known as Amazon **EventBridge**
- **Alarms**
- **Dashboards** \- visualise Cloud Metrics using various graphs
- ServiceLens \- visualise health, performance and availability of app in single place
- Container Insights \- collects and summarises metrics and logs from apps and microservices
- Synthetics \- test web apps to check health
- Contributor Insights \- view top contributors impacting performance of systems

All built off CloudWatch Logs. Layered system.

### CloudWatch Logs

Centralised log-management service which can:

- Export logs to S3
- Stream to ElasticSearch Service (ES) \- to use ELK stack
- Stream CloudTrail Events
- Encrypt logs \- encrypted using SSE at rest by default. Can use own customer-managed key (CMK) via KMS.
- Retain logs \- kept indefinitely by default. Can adjust 1 day \-\> 10 years.

Log groups \- collection of log streams commonly named using forward slash syntax (e.g. ‘/myapp/prod/db’). Log group retention can be set to options between 1 day and 120 months (10 years), or to never expire.

Log streams \- sequence of events from an application or instance being monitored. Can be created manually but usually done by service. Log streams for processes usually named related to the instance ID.

Log events \- single lines in log streams representing a single event. Can be filtered.

### CloudWatch Logs Insights

Enables interactive searching and analysing of CloudWatch log data with more robust filtering and less hassle than exporting to S3 and analysing using Athena. Supports all types of logs. Has its own language called CloudWatch Logs Insights Query Syntax. Single request can query up to 20 log groups, times out after 15 mins and results are available for 7 days. There are sample queries and queries can be saved for later.

### Discovered Fields

When Insights reads a log, it attempts to structure the content by generating fields (starting with ‘@’) which can be used in a query.  
Always generated:

- @message \- raw log event
- @timestamp \- self explanatory
- @ingestionTime \- time log event was received by CloudWatch Logs
- @logStream \- name of the log stream containing the log event
- @log \- log group identifier (account-id:log–group-name)

Others depend on which service logs are generated from.

### Data Availability

AWS Services emit data to CloudWatch on varying intervals based on the service. **EC2 basic monitoring is 5 min**, others can be 1 (usually), 3 or 5 min. Detailed monitoring for EC2 brings this down to 1 min intervals.

### Agent / Host Level Metrics

Some metrics not tracked by default for EC2 instances.  
Default / Host level metrics:

- CPU usage
- Network usage
- Disk usage
- Status checks
  - Underlying hypervisor status
  - Underlying EC2 instance status

Agent level metrics (require installing CloudWatch Agent):

- Memory utilisation
- Disk swap utilisation
- Disk space utilisation
- Page file utilisation
- Log collection

Agent used to collect various locs from EC2 instance and send to a CloudWatch log group.  
Agent can be installed using AWS Systems Manager (SSM). Run command AWS-ConfigureAWSPackage with AmazonCloudWatchAgent name parameter. Must attach relevant role to EC2 instance.

### CloudWatch Alarms

Monitor CloudWatch metric and trigger an action when it breaches the defined threshold.  
States:

- OK \- metric within defined threshold
- ALARM \- metric outside defined threshold
- INSUFFICIENT_DATA \- not enough data available to know

Actions:

- Notification
- Auto-scaling group
- EC2 action

Components:

- Threshold condition \- defines datapoint breach
  - Static \- use value as threshold
  - Anomaly detection \- use a band as a threshold (for when fluctuation is expected e.g. spikes in traffic at 9am)
- Data point \- measurement at given time
- Metric \- what we are measuring
- Evaluation periods \- number of previous periods
- Datapoints to alarm \- number of datapoints out of the evaluation period which need to be breached to trigger the alarm (e.g. 2 out of the last 4\)

Composite alarms are alarms which watch other alarms and effectively combine them into one alarm to reduce noise. Only action for composite alarms is to publish to an SNS topic.

### CloudWatch EventBridge

Fully-managed event bus, connecting separate systems like an MQ might do, but taking the weight of the emitter and consumer by reducing the constraints on the emitter and pushing the event straight to the consumer (considering rules) rather than requiring polling.  
Types:

- Default \- an AWS account has a default event bus
- Custom \- scoped to multiple accounts or AWS accounts
- SaaS \- scoped to third party SaaS providers

Events are JSON objects emitted by services travelling within the event bus. They are made up of:

- Version \- 0 by default
- Id \- unique value for each event
- Detail-type \- identifies fields and values appearing in ‘detail’ field
- Source \- service sourcing event
- Account \- 12-digit AWS account number
- Time \- event timestamp
- Region \- Region event originated
- Resource \- JSON array containing ARNs identifying resources involved in event
- Detail \- JSON object containing data provided by the service (varies)

Producers are AWS services that emit events (even when they are not consumed).

### Scheduled Expressions

EventBridge Rules can trigger on a schedule like serverless cron jobs. Scheduled events use UTC time with minimum precision of 1 minute. Supports cron expressions and rate expressions.

### CloudTrail

Not all AWS services emit CloudWatch events, so CloudTrail is used instead \- allowing EventBridge to track changes to these services made by API calls or users. AWS API call events \> 256 KB are not supported.

### Event Patterns

Filter which events should be used to pass along to targets. Filter events by providing the same fields and values found in the original events.  
Matching types:

- Prefix \- match on prefix of a value in the event source
- Anything-but \- match anything except what is provided in the rule
- Numeric \- match on numeric comparisons
- IP address \- match against IPv4/6 addresses
- Exists \- match on presence or absence of field in JSON
- Empty value \- match “” for strings or null for other types
- Complex / multiple \- combine matching rules above into complex pattern

### EventBridge Rules

Specify up to 5 targets for a single rule. Possible targets include: Lambda functions, SQS queues, SNS topics and much more. Specify what gets passed along using ‘Configure input’ setting \- acting like a filter. Can have 100 rules per bus.  
Configure input option types:

- Match events \- pass on the entire event pattern text (everything)
- Part of matched event \- just a specified part of the event text (e.g. “\$.detail”)
- Constant (JSON text) \- Send static content (e.g. ‘{“success”:true}’)
- Input transformer \- Map fields from event data to variables and insert those into strings or JSON (e.g. ‘{“state”:”\$.detail.state”}’ where “the state is \<state\>”)
  - Cannot use (reserved):
    - aws.events.rule-arn
    - aws.events.rule-name
    - aws.events.event

### Partner Event Sources

A list of third-party service providers can be integrated to work with EventBridge \- with events emitting from these service providers and going into the event bus.

### Schema Registry

Allows creation, discovery and management of OpenAPI schemas for events on EventBridge. Schema is an outline, diagram or model used to describe the structure of different types of data.  
Why:

- See if structure of events has changed over time (versioning)
- Easier for developers to know what data to expect from a type of event
  - Can download Code Bindings to make it easy to work with events in code
    - Wraps schema in programming object

## Amazon Kinesis

Fully-managed solution for collecting, processing and analysing streaming data in the cloud. For real-time, think Kinesis.  
Streaming data examples:

- Stock prices
- Game data
- Social network data

Types of Kinesis stream:

- Kinesis Data Streams
  - Configure custom producers (sending data to the stream)
    - Amazon Kinesis Agent \- standalone Java app that will monitor files based on a pattern and send the data to Kinesis when it changes
    - AWS SDK \- simple way to publish, does not scale
    - AWS Direct Integration \- Aurora, CloudFront, DynamoDB etc.
    - Amazon Kinesis Producer Library (KPL) \- java-only library allowing publishing data to a data stream at scale. Use when:
      - Sending multiple records /s
      - Scaling producer vertically by 100x
      - Requiring a producer highly efficient in underlying compute resources
  - Configure custom consumers (receiving data from the stream)
    - Third Party \- other data streams / data processing frameworks like Apache Fink, Kafka Connect etc.
    - Kinesis Data Firehose \- in turn integrates delivery to other AWS services
    - AWS SDK \- simple way to read from stream, not scalable
    - Amazon Kinesis Client Library (KCL) \- java library allowing creation of custom consumer (multi lang supported here)
  - Shards \- partitioned compute to which stream data is written
    - Up to 5 transactions /s reads
    - Up to max total data read rate of 2MB/s
    - Up to 1000 records /s writes
    - Up to max total data write rate of 1MB/s (including partition keys)
    - Each shard has sequence of data records
      - Each record has sequence number assigned by Data Stream
    - Increase/decrease number of shards allocated
    - Must be iterated in gets
  - Partition Keys \- group data by shard within a stream
    - Unicode strings
    - Max length 256 chars
    - MD5 hash function used to map keys to 128-bit ints and map associated data records to shards
    - When an app puts data into a stream, partition key must be specified
  - Sequence Number \- identify data records per partition within shard
    - Unique per partition key within shard
    - Generally increase over time for same partition key
    - Longer time periods between write requests \= larger sequence numbers
  - Most flexible data streaming option
  - Capacity modes:
    - On demand
      - For unpredictable workloads
      - Automatically scales
      - Pay based on data ingested and retrieved
      - 2 consumers by default (enhanced fan-out (EFO) to add 20 more)
      - Very similar to Data Firehose but with more customisation
    - Provisioned
      - For predictable workloads
      - Customer manages shards
      - Pay based on number of shards and data transfer
      - Max 200 shards
  - Can switch between capacity modes at any time
  - Retention period \= 24 hours by default
    - Can be changed to up to 365 days
    - Takes several minutes to come into effect and incoming records will follow old retention during this time
- Amazon (Kinesis) Data Firehose
  - Serverless and simpler version of data streams
    - 1 consumer
    - Data immediately disappears once consumed
  - Direct integration with specific AWS services
  - Pay on demand based on consumed data
  - Can convert incoming data to other formats and compress / secure data
    - Convert to Parquet or ORC (use Amazon Glue table)
    - Can transform using Lambda
      - Must return record id, result and data
  - Consumers / destinations:
    - AWS Services
      - S3
      - Redshift
      - Etc.
    - Custom:
      - HTTP Endpoint
    - Third Party:
      - Splunk
      - Etc.
  - Dynamic Partitioning enables continuously partitioning streaming data using keys within data
    - Data delivered grouped by keys into S3 prefixes
    - Makes it easier to run high performance analytics on streaming data
    - Inline partitioning using a JQ expression or Lambda function to parse data
    - Cannot be turned off once enabled on a stream
- Managed Service for Apache Fink (formerly: Amazon Kinesis Data Analytics)
  - Run queries against data flowing through real-time stream
  - Create reports and analysis on emerging data
  - Use custom SQL
- Kinesis Video Streams
  - Analyse or process real-time streaming video
  - Can go to ML consumers

### Enhanced Fan Out (EFO)

Allows up to 20 consumers to receive records from a stream with throughput of up to 2 MB /s per shard. Consumers utilising EFO have dedicated throughput per consumer. Consumers must be configured using KCL or Streams API to utilise EFO.

## AWS CloudTrail

Monitors API calls and actions performed on an AWS account \- enabling governance, compliance and auditing of AWS accounts. Provides who, what, when and where of all actions. On by default and logs are preserved for the last 90 days via Event History. If longer than 90 days needed, create a Trail which outputs to S3 and requires Amazon Athena to analyse efficiently (CloudTrail Lake uses Athena under the hood). Commonly forwarded to CloudWatch logs.

### Traces & Spans

Useful in microservice systems.  
A **trace** is a data/execution path through a system and can be thought of as a directed acyclic graph (DAG) of spans.  
A **span** represents a logical unit of work that has an operation name, start time of operation and duration of operation. Spans may be nested and ordered to model causal relationships.

![][image3]

### Open Telemetry (OTEL)

Collection of open-source tools, APIs and SKDs to instrument, generate, collect and export telemetry data. Standardised the way telemetry data (metrics, logs and traces) are generated and collected.  
Terms:

- Instrumentation \= embedding a monitoring library into an application to capture monitoring data like metrics, traces or logging.
- Collector \= agent installed on target machine or as dedicated server which is a vendor-agnostic way to receive, process and export telemetry data
  - Removes need to run, operate and maintain multiple agents
  - Local collection agent is default export location for instrumentation libraries

### AWS Distro for OpenTelemetry (ADOT)

Secure AWS-supported distribution of OpenTelemetry. Send correlated logs, metrics and traces to or from observability backends:

- Amazon Managed Service for Prometheus (AMP)
- Amazon managed Streaming for Apache Kafka (MSK)
- Amazon CloudWatch
- AWS X-Ray
- Amazon Open Search
- Any OpenTelemetry Protocol (OTLP) compliant backend

Observe apps running in:

- EC2
- ECS EC2
- Fargate
- EKS
- AWS App Runner
- AWS Lambda
- On-premise

### Prometheus

Open-source systems monitoring and alerting toolkit originally built at SoundCloud.

- Collects and stores metrics as time series data \- time series database
- Main features:
  - Multi-dimensional data model identified by metric name and key/value pairs
  - PromQL \- flexible query language
  - No reliance on distributed storage
    - Single server nodes are autonomous
  - Time series collection happens via a pull model over HTTP
    - Pushing time series supported via an intermediary gateway
  - Targets discovered via service discovery or static configuration
  - Multiple modes of graphing and dashboarding support
- Values reliability
  - Can always view what statistics are available about a system \- even under fail conditions
- If 100% accuracy required (e.g. per-request billing), Prometheus not a good choice
  - Collected data will not be detailed and complete enough
  - Would need another system to collect and analyse data for billing

How it works:

- Scrapes metrics from instrumented jobs \- directly or via intermediary push gateways for short-lived jobs
- Stores all scraped samples locally
- Runs rules over this data to aggregate and record new time series from existing data or generate alerts
- Grafana or other API consumers can be used to visualise

#### Amazon Managed Service for Prometheus (AMP)

Prometheus-compatible monitoring service for container infrastructure and application metrics \- fully-managed Prometheus server environment. AMP makes it easy to securely monitor container environments at scale.

- Can use Amazon Managed Service for Grafana (AMSG) to visualise data within AMP
  - Fully-managed and secure service to instantly query, correlate and visualise operational metrics, logs and traces from multiple sources
- Can use AWS Distro for OpenTelemetry (ADOT) to ingest application metrics from an environment with AMP

## AWS Audit Manager

For continually auditing AWS usage to simplify risk and assess compliance.  
Contains:

- Framework library
  - Browse library of control frameworks that support various compliance standards
    - e.g. PCI, CIS, SOC2, HIPAA etc.
- Control library
  - Browse existing controls or create custom ones to use in a custom framework

Can create assessments to review evidence collected and generate assessment reports. Can continuously collect evidence by integrating AWS Config and AWS Security Hub.  
Similar to Security Hub but has itemised assessments & evidence.

## AWS Inspector

Hardening \= eliminating as many security risks as possible. Hardening in VMs involves running a collection of security checks known as security benchmarks.

AWS Inspector runs a security benchmark against specific EC2 instances. It can run a variety of security benchmarks and both Network and Host assessments.

Steps:

1. Install AWS SSM agent on EC2 instance
2. Add relevant permissions
3. Run an assessment on the target
4. Review findings and remediate security issues found

A popular benchmark is by CIS and has 699 checks.

There are also passive scans.

## Amazon Macie

Fully-managed service that continuously monitors S3 data access activity for anomalies and generates detailed alerts when it detects risk of unauthorised access or inadvertent data leaks.  
Works by using machine learning to analyse CloudTrail logs.  
Alerts:

- Anonymised access
- Config compliance
- Credential loss
- Data compliance
- File hosting
- Identity enumeration
- Information loss
- Location anomaly
- Open permissions
- Privilege escalation
- Ransomware
- Service disruption
- Suspicious access

Macie will identify the users most at-risk of leading to a compromise.

## AWS Security Hub

Cloud Security Posture Management (CSPM), allowing generation of a security score to determine security posture. Allows enabling of standards \= collections of security controls \= AWS Config rules.

### Amazon OpenSearch

Managed full-text search service making it easy to deploy, operate and scale OpenSearch (a popular open-source search and analytics engine).  
Two engines can be deployed:

- OpenSearch
  - Open source fork of ElasticSearch and Kibana
- ElasticSearch
  - Search engine based on Lucene library
  - Free and open (for non-enterprise?)

ELK stack is ElasticSearch, Logstash and Kibana \- commonly used together for logs and analytics purposes.  
Can be deployed:

- On provisioned clusters
- Serverless \- automatically provisions and scales dedicated compute
