### Amazon Quantum Ledger Database

Amazon QLDB reached end of support on 31 July 2025 and is no longer available.

## ElastiCache

Fully-managed in-memory datastore for Valkey, Memcached or Redis OSS, intended to cache data or HTML fragments to improve response times down in the 10s-100s of milliseconds.  
ElastiCache:

- Only accessible by resources in same VPC (serverless caches can optionally expose a public endpoint)
  - To ensure low latency for cache purposes
- Can be deployed in multiple AZs for high availability
- Can be deployed on-premise via AWS Outposts (standard)
- Can be replicated cross-region via ElastiCache Global Datastores
- Can automatically perform backups of data stores
- Nodes can be reserved to save money using ElastiCache standard
- Can use RBAC for Redis 6.0+ to manage user access via Management Console

Deployment options:

- Standard
  - Use for predictable workloads
  - Customer managed cluster and nodes
  - Billed based on number and type of nodes
- Serverless
  - Use for unpredictable workloads
  - Automatically scale
  - Billed on data stored and ElastiCache Processing Units (ECPUs)

Caching types:

- Memcache
  - Open-source caching layer for web-apps
  - Generally preferred for caching HTML fragments
  - Simple key/value store
    - Strings
  - Very fast but very basic
- Redis
  - Open-source in-memory database store
  - Key / value store
    - Strings
      - Binary safe so can contain images etc.
      - Max length 512MB
    - Sets
      - Unordered
      - Unique items \- no duplicates
    - Sorted sets
      - Ordered by associated scorers
      - Unique items
    - Lists
      - Ordered
      - Non-unique items \- can have duplicates
    - Hashes
      - Mappings between string fields and string values
    - More
  - Very fast database
  - Volatile as data stored in-memory

## Amazon MemoryDB

Valkey and Redis OSS-compatible in-memory database for fast performance. Main difference from ElastiCache is MemoryDB has persistence guarantees making it **suitable as a primary database**. Microsecond reads and single-digit millisecond writes as a trade-off. Billed per usage.

## Amazon Redshift

Fully-managed petabyte-scale data warehouse which can be used to analyse massive amounts of data using complex SQL queries. Columnar store database (stores data for whole columns rather than rows) \- reduces overall disk IO requirements for loading data (important for optimising analytical performance). Cheap compared to similar services. Also good compression due to column storage.  
Database vs. Data warehouse:

- Database
  - Online Transaction Processing (OLTP)
  - Built for short and fast transactions
  - E.g. adding an removing items from a shopping list
- Data warehouse
  - Online Analytical Processing (OLAP)
  - Built for long and complex queries across data
  - Often multiple sources
  - E.g. generating reports on large data

Example use:  
Want to continuously copy data from EMR, S3 and DynamoDB into one place to analyse for a custom business intelligence tool. Would use a third-party library to connect and query Redshift for data.

Configurations:

- Single node \- 160GB (dc2.large)
- Multi-node
  - Leader node \- manages client connections and receiving queries
  - Compute nodes \- store data and perform queries
    - dc2.large: 1-32 nodes (160GB each); larger node sizes (e.g. dc2.8xlarge) support up to 128 nodes

Node types:

- RA3 / RG \- managed storage (local SSD plus S3), compute and storage scale independently (recommended)
- DC2 \- local SSD, best for datasets under 1 TB
- DS2 is no longer available
- Start at large \- no smaller

Redshift uses Massively Parallel Processing (MPP), which automatically distributes data and query loads across all nodes.  
Backups enabled by default with 1 day retention (can be set up to 35 days). Redshift always maintains at least 3 copies of data:

- Original copy
- Replica on compute nodes
- Backup copy in S3

Can asynchronously replicate snapshots to S3 in different regions.  
Billed for compute node hours (not leader node), backup storage and data transfers.  
Data encrypted in transit and at rest. Via KMS or CloudHSM.  
Redshift is **single-AZ** by default. Multi-AZ is available for RA3/RG provisioned clusters.

Amazon Athena  
Interactive query service for analysing data from S3. Has Athena SQL which runs SQL queries on S3 buckets (often dumping results to S3 bucket too) and Apache Spark which interactively runs data analytics. Serverless \- only pay for usage.  
Integrates with:

- CloudFormation
- CloudFront
- CloudTrail
- DataZone
- ELB, EMR, AWS Glue Data Catalog
- IAM
- QuickSight
- S3 Inventory
- Step Functions
- Systems Manager Inventory
- VPC

Athena SQL components:

- Workgroup \- saved queries which other users can be granted access to
- Data source \- group of available databases (catalog)
- Database \- group of tables (schema)
- Table \- data organised as group of rows / columns
  - Can be created using SQL CREATE or AWS Glue Wizard
- Dataset \- raw data of the table

SQL subsets:

- Data Definition Language (DDL)
  - Defines schema
  - E.g. CREATE, ALTER, DROP
- Data Manipulation Language (DML)
  - Manipulates datasets
  - E.g. INSERT, UPDATE, DELETE
- Data Query Language (DQL)
  - Selects datasets
  - E.g. SELECT

SerDe \- serialisation / deserialisation libraries for parsing data from different data formats (e.g. CSV, JSON, Parquet, ORC). SerDe defines the table schema \- not the DDL.

## AWS Data Exchange

Catalogue of third-party datasets which can be downloaded for free, an upfront cost, or usage based/subscription cost. Datasets can be uploaded and sold by anyone but there are strict checks done. Data grants allow controlled access to your datasets. Open Data on AWS is a collection of 300+ free datasets.

## AWS Glue

Serverless data integration service making it easy to discover, prepare, move and integrate data from multiple sources. You can discover and connect to more than 70 diverse data sources and manage the data in a centralised data catalogue.  
Use cases:

- Analytics
- Machine learning
- Application development

Visually create, run, monitor, extract, transform and load (ETL) pipelines to load data into data lakes.  
Immediately search and query catalogued data using:

- Amazon Athena
- Amazon EMR
- Amazon Redshift Spectrum

Capabilities:

- Data discovery
- Modern ETL or ELT
- Cleansing
- Transforming
- Centralised cataloging

### AWS Glue Jobs

Engines for glue jobs:

- Python Shell Engine
- Ray
- Spark

Can be created in:

- Visual ETL (AWS Glue Studio)
- Jupyter Notebooks
- Script Editor (within AWS)

Charged based on number of data processing units (DPUs):

- 10 DPUs to each Spark job
- 2 DPUs to each Spark Streaming job
- 6 DPUs to each Ray job
- Combination of worker type and number of workers determines DPUs

### AWS Glue Studio

Visually build ETL (Extract, Transform, Load) / ELT (extract, load, transform) pipelines.  
Pipelines are made up of connected nodes:

- Sources \- the data you plan to use
- Transforms \- what you want to do to the data
- Targets \- where you want to send the data

You can version control pipelines using:

- AWS Code Commit
- GitHub
- GitLab
- BitBucket

Can build just with code. Visual mode will generate a script you can use.

### Glue Data Catalogue

Fully-managed Apache Hive Metastore-compatible catalogue service for customers to store, annotate and share metadata about data. Serverless so pay for usage.  
Integrates with:

- S3
- RDS
- Redshift
- Athena
- Glue ETL
- EMR

Table formats:

- Standard AWS Glue table
  - Specify data format
    - Avro
    - CSV
    - JSON
    - XML
    - Parquet
    - ORC
  - Data can be sourced from:
    - S3
    - Kinesis
    - Kafka
- Apache Iceberg table
  - Uses own expressive SQL data format

Terms:

- AWS Glue database \- container for multiple Glue tables
- AWS Glue table \- metadata definition representing data (including schema)
  - Can be used as a source or target in job definitions
- AWS Glue Data Crawler \- analyse targeted data source to determine its schema and generate Glue Data tables
  - Can be connected to:
    - S3
    - Java Database Connectivity (JDBC)
      - Redshift
      - Snowflake
      - RDS
    - DynamoDB
    - MongoDB client
      - MongoDB server, MongoDB Atlas, DocumentDB
    - Delta Lake
    - Apache Iceberg tables stored in S3
    - Hudi tables stored in S3
  - Can be run:
    - On schedule
    - On demand

## AWS Lake Formation

Data lake to centrally govern, secure and globally share data for analytics and machine learning.  
Permissions:

- Manage fine-grained access control for data lake data on S3
- Manage metadata in AWS Glue Data Catalog
- Provides its own permissions model that augments the IAM model through simple grant/revoke mechanism similar to relational database management system (RDBMS)
- Allows sharing data internally and externally across multiple AWS accounts, organisations or directly with IAM principles in another account
- Permissions enforced using granular controls at column, row and cell levels

Integrates with:

- Athena
- Quicksight
- Redshift Spectrum
- EMR
- Glue

Lake formation and Glue share the same data catalog.

### Data Lakes

Centralised data repository for large quantities of unstructured or semi-structured data. Generally store data in object (blobs) or file mediums.  
You can:

- Collect \- pull data from various sources
- Transform \- change or blend data into new semi-structured data using ELT/ETL engines
- Distribute \- allow access to data for various programs / APIs
- Publish \- publish datasets to meta catalogs

## RDS (Relational Database Service)

Managed (not fully) service for multiple relational databases featuring:

- Various db engine support
- Automatic and manual backups
- Multi-AZ
- Read replicas
- Performance insights
- Customisable db params
- RDS proxy for a connection pooler
- Various authentication methods
- Blue/green deployment
- Etc.

Database Engines:

- MySQL
  - Open-source SQL database
  - Owned by Oracle
  - Replication and partitioning features for scalability and availability
- MariaDB
  - Fork of MySQL when Oracle purchased it
  - Open-source
  - Highly compatible with MySQL
- Postgres
  - Open-source SQL database
  - More feature-rich than MySQL
    - Also more complex
  - Supports advanced data types and functions
    - JSON, XML, key-value pairs
- Oracle
  - Oracle’s SQL database for enterprises
  - Need a license
  - Complex architecture supporting large scales
- Microsoft SQL Server
  - Microsoft’s SQL database
  - Need a license
  - Integrates with other Microsoft products and services
- IBM DB2
  - IBM’s SQL database
  - Need a license
  - High-performance and scalability in large environments
- Amazon Aurora
  - Fully-managed AWS service
  - Compatible with MySQL and Postgres
  - Automatically divides database volume into 10GB segments across many disks
    - Enhances performance and reliability

Encryption:

- Encryption at rest
  - Available for all RDS engines
  - Must be turned on
  - Will encrypt all automated backups, snapshots and read replicas
  - Encryption handled by KMS
  - Can only be turned on during creation
    - Snapshots can be taken and new instances launched with encryption on to help transition
- Encryption in transit
  - SSL/TLS is supported on all engines; clients must connect using TLS. Enforcement is engine-specific and can be configured via parameter groups (e.g. `rds.force_ssl`)

Backups:

- Automated backups
  - No additional charge
  - Creation will take longer due to snapshot creation
  - Choose retention period 0-35 days
    - 0 \= off
    - Point-in-time recovery (PITR) can restore at any 5 min interval within retention period
  - Stores transaction logs throughout the day
  - Enabled by default
  - All data stored in S3
  - Storage I/O may be suspended during backup
- Manual backups (snapshots)
  - Backups exist even if original RDS instance deleted
  - You can:
    - Copy snapshots across regions
    - Share snapshots to other AWS accounts
    - Export snapshots to S3
  - Additional storage costs
  - Database has to be in ‘available’ state
- Restoring backups creates a new RDS instance and restores the data onto that instance (slow)

Subnet Group:

- Collection of subnets (usually private subnets) that you create in a VPC and then designate for DB instances
- Each db subnet group should contain subnets in at least 2 AZs in any given region
- RDS chooses a subnet from the subnet group to deploy RDS instance to
- Subnets in a DB subnet group are either public or private
  - If any of the subnets are private, it’s private

Multi AZ:

- When you have standby RDS clusters or instances in another AZ which fail over if an AZ becomes unavailable
  - Multi AZ DB instance deployment: one non-readable standby (RDS)
  - Multi AZ DB cluster deployment: two readable standbys in 3 AZs (RDS, supported engines only)
  - Aurora clusters are different: shared storage across 3 AZs
- Creates an exact copy of your db and data in another AZ
- AWS automatically synchronises changes in the db over to the standby instance
- When AZ failure occurs, the standby instance is promoted to primary
- Apply immediately on an existing instance or you’ll have to wait until the next maintenance window

Read Replicas:

- Run multiple read-only copies of a database
- Improves read contention, improving performance and latency
  - Read contention is multiple processes or instances competing for access to the same index/data block at the same time
- Must have automatic backups enabled
- Replication is asynchronous between primary and replicas
- Up to 15 replicas per source for RDS (MySQL, MariaDB & PostgreSQL dbs)
- Up to 15 replicas for Aurora
- Each read replica has its own DNS endpoint
- Replicas use the same storage type as the source db by default
  - Can be changed
- Can have:
  - Multi-AZ replicas
  - Replicas in another region
  - Replicas of replicas
- Replicas can be promoted to their own db
  - This breaks replication
  - If primary fails, must manually update URLs to point at copy

| Feature              | MySQL / MariaDB        | Oracle                  | Postgres                | SQL Server              |
| -------------------- | ---------------------- | ----------------------- | ----------------------- | ----------------------- |
| Replication method   | Logical representation | Physical representation | Physical representation | Physical representation |
| Writable             | Yes (can be enabled)   | No                      | No                      | No                      |
| Manual backup        | Yes                    | Yes                     | Yes                     | No                      |
| Automatic backup     | Yes                    | Yes                     | No                      | No                      |
| Parallel replication | Yes                    | Yes                     | No                      | Yes                     |

Multi AZ vs. Read Replicas:

|                        | Multi-AZ DB Instance Deployments                     | Read Replicas                                 |
| :--------------------- | ------------------------------------------ | --------------------------------------------- |
| **Replication method** | Synchronous (durable)                      | Asynchronous (scalable)                       |
| **Active**             | Only primary instance is active            | All replicas are active for read              |
| **Backups**            | Automated from backups too                 | None by default                               |
| **Scope**              | Always span 2 AZs in single region         | Can be within an AZ, cross-AZ or cross-region |
| **DB engine upgrades** | Happen on primary                          | Independent from source instance              |
| **Promotion**          | Automatic failover when problem in primary | Manually promoted to standalone db instance   |

DB Instances:

- Isolated database environments running in the cloud
- Contain one or more user-defined databases
- Up to 40 Amazon RDS DB instances per AWS account
  - Depends on db engines and license models
- Each db has user-defined database **instance** identifier and AWS-defined unique instance identifier as part of the DNS hostname
  - E.g. ‘https\://**my-rds-instance**._mnopqrstuvwx_.us-west-1.rds.amazonaws.com’
- Classes:
  - Determine available compute and memory available
  - General purpose
    - db.m-
  - Memory-optimised
    - db.x-, db.z-, db.r-
  - Burstable performance
    - db.t-
  - Optimised reads
    - Db.r-
- Storage:
  - Can use:
    - General purpose SSD
    - Provisioned IOPS SSD
    - Magnetic (not recommended)
  - Max storage of most instance classes \= 64TB
  - Can be increased
    - Not decreased (would have to spin up new instance with less storage)

RDS Performance Insights helps identify bottlenecks and performance issues. Turned on by default, providing 1 week of data. Retention period can be changed up to 2 years for additional cost.

RDS Custom:

- Allows customers to directly manage aspects of RDS maintenance instead of AWS
  - Install third-party applications
  - Install custom patches
  - Create own automation
- How:
  - Create RDS Custom DB instances
  - Connect an RDS Custom DB instance endpoint
  - Directly access the host to make changes
- Works with:
  - Microsoft SQL server
  - Oracle database

RDS Proxy:

- Creates a connection pooler so that short-lived AWS Lambda functions connecting to RDS don’t exhaust the connection limit e.g.
  - RDS instance with connection limit of 20
  - 50 Lambda functions that could fire any time and open a connection
  - RDS Proxy goes in the middle
    - Creates 20 connections and keeps them open, reusing them for Lambdas trying to start a connection
    - This does not magically allow more concurrent connections but does reduce overhead of opening and closing connections
    - Lambdas can stay connected to the RDS proxy (just not the RDS instance itself) while their connection is freed up for another client

Optimised reads and writes:

- Allow faster read and write operations for improved performance
- Uses NVMe-based SSD block storage instead of EBS for temporary tables to achieve this
  - Queries using temporary tables:
    - Sorts
    - Hash aggregations
    - High-load joins
    - Common Table Expressions (CTEs)
- Available for specific combinations of instance class and engine version
  - Some db engines only allow optimised reads
  - Different requirements for reads and writes
  - Additional db config may be required

Authentication  
IAM:

- Authenticate with an RDS instance’s db using IAM authentication instead of a password
- Works with:
  - MySQL
  - MariaDB
  - Postgres
- Each token has a 15 min lifetime
- Can use standard authentication alongside IAM
- Both users and EC2 instances can do this
- Process:
  - Enable on RDS instance
  - Create policy and attach to user or role to allow to connect as user
  - Create user on db
  - Generate auth token to be used when connecting

Kerberos:

- Network authentication protocol directly integrated into Microsoft Active Delivery
- Allows for SSO
- Works with:
  - AWS Directory Service for Microsoft Active Directory
  - On-premise Active Directory
- Works with:
  - Microsoft SQL Server
    - Support one and two-way forest trust relationships
  - Postgres
    - Support one and two-way forest trust relationships
  - MySQL
  - Oracle
    - Support one and two-way **external** and forest trust relationships

AWS Secrets Manager:

- Can manage an RDS instance’s master user password
  - Allows rotation
- Does not work with:
  - Microsoft SQL Server
  - Amazon RDS blue/green deployments
  - Amazon RDS Custom
  - Oracle Data Guard switchover
  - RDS for Oracle with CDB
- Secret rotated every 7 days by default
- Web-apps need to be configured to access the password from Secrets Manager
- If db is deleted, secret is too
- Costs

Master User Account:

- The initial database account created when the db instance is provisioned
- Has full administrative privileges on the db
- Not recommended for daily use
  - Create users with least privilege possible to perform duties
- Password can be reset

Database Activity Streams:

- Allows controlling administrator access to data streams
- Must be turned on (not on by default)
- RDS pushes activities to Amazon Kinesis data stream
  - Created automatically
  - Can monitor activity from here or consume the activity stream with other services / applications

Parameter Groups:

- Act as containers for engine configuration values applied to one or more db instances
- Each database engine will have completely different database parameters
- Alter these to suit your config
- If you need more configurability use RDS Custom

Public Accessibility:

- Turn on with –publicly-accessible option
- Determines whether the DNS Endpoint will resolve to the private IP address from traffic outside of the VPC
- Does not override Security Group rules
  - Must allow inbound traffic on specific db ports
- Useful to connect to RDS instance without having to use intermediate way of accessing the db

Public connections can be made to a db:

- Via the DNS endpoint (connecting directly using a db client or driver)
  - Connection url string gives all the data needed in one string
    - Protocol
    - Hostname
    - Port
    - Database name
    - Username
    - Password
  - Default ports:
    - MySQL \= 3306
    - Postgres \= 5432
    - Oracle \= 1521
    - SQL Server \= 1433
    - Aurora \= same as standard for that engine
- Via a public web server
  - Generally better practice
    - Can have things like connection pooling

Private connections can be made to a db:

- Via a Cloud9 server in a public subnet in the same VPC
- Via a web server EC2 instance in a public subnet in the same VPC
- Via a Bastion or Jumpbox tunnelling through
- Using AWS Client VPN to connect your machine to the VPC and establishing a connection
- On-premise using AWS Direct Connect from on-premise network
- CloudShell **cannot** be used for private connection as it doesn’t reside in customer managed VPC

RDS Blue/Green Deployments:

- Copies production database environment into a separate synchronised staging environment
- Database changes are tested here without risking the prod
  - Database patches / system updates
  - New database features
- Different database engines will have different prerequisites

Extended support:

- Allows running db on a major engine version past the RDS end of standard support date
- Up to 3 years
- Costs
- Amazon will supply patches for ‘critical’ and ‘high’ CVEs

## Aurora

Fully-managed relational database cluster.  
Can run:

- Aurora MySQL
  - 5x better performance than traditional MySQL
- Aurora Postgres
  - 3x better performance than traditional Postgres
- At 1/10th cost of similar fully-manage solutions

Contains most of the other features of RDS plus its own exclusive features.  
Differences to RDS:

- More managed
- More instances

Attributes:

- Durability / fault tolerance
  - Aurora Backup and Failover are handled automatically
  - Snapshots of data can be shared with other AWS accounts
  - Storage is self-healing
    - Data blocks and disks continuously scanned for errors and repaired
- Availability
  - Deploys in minimum of 3 AZs
  - Each contains 2 copies of data at all times
  - Lose up to at least 2 copies of data without affecting **write** availability
  - Lose up to at least 3 copies of data without affecting **read** availability
- Storage
  - Cluster starts with 10GB storage
  - Scales up in 10GB increments up to 64TB / 128TB depending on db engine versions
  - Storage auto-scales
  - Computing resources scale up to 32 vCPUs and 244GB memory
- Security
  - TLS / SSL certificates can be applied to encrypt secure connections so termination occurs at database
  - Data encrypted at rest by default
    - Cannot be turned off
    - Can use KMS keys

Provisioned is the traditional/default compute configuration for Aurora. Aurora db cluster contains:

- A primary db instance that performs reads and writes
  - Not created by default like it is in RDS
- Up to 15 Aurora Replicas (read db instances) (optional)

Reader & Writer Instances

| Attribute         | Reader                                    | Writer                                      |
| ----------------- | ----------------------------------------- | ------------------------------------------- |
| Role              | Just reads                                | Writes & reads                              |
| Quantity          | 0-15 per cluster                          | 1 per cluster                               |
| Scalability       | Horizontal                                | Vertical only                               |
| Availability      | Distributed reads, can be failover target | Critical \- failure causes failover         |
| Use Cases         | Read-heavy workloads & analytics          | Transactional changes                       |
| Failover Capacity | Can be promoted to writer                 | Automatic promotion of a reader when failed |
| Cost              | Increases with each instance              | Based on instance size and IOPS             |

Aurora Serverless v2:

- Fully-manages autoscaling configuration for Aurora
- Capacity adjusted automatically based on demand
- Charged for resources the db clusters consume
- Can scale down to 0 ACUs and automatically pause when idle if minimum capacity is set to 0 (newer engine versions; otherwise minimum is 0.5 ACUs)
  - Aurora Capacity Units (ACUs) determine cost vs capacity
  - 1 ACU is about 2GiB memory, CPU and networking

Serverless vs. Provisioned

| Attribute          | Serverless                                                   | Provisioned                                      |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------ |
| Scaling            | Fine-grained, almost instant scaling                         | Manual scaling. Requires planning & downtime     |
| Capacity range     | 0-256 ACUs (version dependent) \- flexible                  | Fixed, based on instance size chosen             |
| Scaling speed      | Seconds                                                      | N/A (manual intervention required)               |
| Read/Write scaling | Independent                                                  | Depends on instance type and read replica config |
| Compatibility      | Broader version support                                      | Wide version support, depending on instance type |
| Use cases          | Highly variable workloads needing frequent/immediate scaling | Stable/predictable workloads                     |
| Billing            | ACUs per second                                              | Instance hours & storage                         |
| Start/stop         | Responsive                                                   | Manual                                           |
| Maintenance        | Minimal downtime                                             | Scheduled maintenance windows                    |

Global Database:

- An Aurora database spanning multiple regions for global low-latency and high availability
- Has up to 10 secondary db clusters in different regions
- Write operations only occur on the primary cluster
- Data replicated to a secondary cluster
  - Typically under a second
- Global db only available in specific regions and specific db versions
- To make:
  - Create global cluster
  - Create primary cluster, placing in global cluster
  - Create secondary cluster, placing in global cluster
  - Create instances after this

RDS Data API:

- Allows the use of HTTP requests to securely query an Aurora database
- Unlimited requests per second
- Must be enabled on cluster to the user
- Data API calls are excluded by default from CloudTrail since they are data events
- Multi statements aren’t supported
- Can’t retrieve multi-dimensional arrays for a query’s column
- Supports specific data types
- Supports execution and transaction statements
- Aurora In the AWS Management Console has a query editor \- this is just an interface to connect via the RDS Data API

Babelfish:

- Open source library for PostgreSQL to understand queries from applications written for Microsoft SQL Server
- Extends Aurora PostgreSQL db clusters with ability to accept db connections from Microsoft SQL
- Apps originally built for SQL Server work directly with Aurora PostgreSQL with few code changes compared to a full migration
  - Also don’t need to change db driver
- Runs Transact-SQL (TSQL)
- Does not support:
  - RDS Blue/Green Deployments
  - AWS IAM
  - Database Activity Streams (DAS)
  - PostgreSQL logical replication
  - RDS Data API
  - RDS Proxy
  - Salted Challenge Response Authentication Mechanism (SCRAM)
  - Query editor
  - Kerberos authentication via Active Directory

## DocumentDB

NoSQL document database that is MongoDB-compatible (basically mongo but AWS getting around licensing issues).

- Cluster types:
  - Instance-based cluster \- manage instances directly choosing instance type
  - Elastic cluster \- clusters automatically scale, choose vCPU and number of instances per shard
- Does not support all functionality in MongoD\~B
  - E.g. writable retries not supported
- Grows in storage volume by increments of 10GB up to 128TB
- Create up to 15 replicas
- Continuously monitors health of cluster
  - Automatically restarts failed instances
  - Failover automatically to up to 15 replicas in other AZs
- Backup turned on by default
  - Cannot be turned off
  - Retention period 1-35 days
  - Supports point-in-time recovery
  - Not sure of precision
- Clusters deployed into customer’s VPC
- Performance Insights feature to determine bottlenecks for reads/writes
- In-transit and at-rest encryption
  - Must connect via TLS

### Document Store/Document Database

A NoSQL database storing documents as its primary data structure.

- Documents could be XML but usually JSON (or JSON-like)
- Document stores are sub-classes of Key/Value stores
- Component terms (relational database \-\> document database)
  - Table \-\> Collection
  - Row \-\> Document
  - Column \-\> Field
  - Index \-\> Index
  - Join \-\> Embedding / Linking

### MongoDB

Open-source document database storing JSON-like documents.

- Primary data structure for MongoDB is BSON (binary JSON):
  - More space efficient than JSON
  - More scan-speed efficient than JSON
  - More data types than JSON
- Use interactive shell (mongosh) or a mongoDB driver to interact
  - Traditionally doesn’t use SQL but there is MQL and Atlas SQL
- Default port \= 27017
- Supports searches against:
  - Fields
  - Ranged queries
  - Regular expressions
- Supports primary and secondary indexes
- High availability achievable via replica sets
- Scales horizontally using sharding
- Can be used as file system (GridFS)
  - Load-balancing and data replication features over multiple machines for storing files
- Ways to group data during a query (aggregation):
  - Aggregation pipeline
  - Map reduce
  - Single purpose aggregation
- Supports fixed-size collections called capped collections
- Claims to support multi-document ACID transactions
  - Atomicity \- all operations in transaction as one unit (no partial execution)
  - Consistency \- maintains valid states, following rules, data types and constraints
  - Isolation \- concurrent operations are separate and simultaneous actions do not interfere with one another
  - Durability \- once operations are complete they are permanent

## DynamoDB

NoSQL key/value and document db for scale applications.  
NoSQL \= a database that is not relational and does not use SQL to query data for results.

- Key/Value storage \= form of storage containing just keys and associated values
- Document store \= nested data structure

Features:

- Fully-managed
- Multi-region
- Multi-master
- Durable database
- Built-in security
- Backup and restore
- In-memory caching
- Capacity modes \- on-demand (pay per request, no capacity planning, unpredictable workloads) or provisioned (specify RCU/WCU, predictable workloads)

Reads:

- Eventually consistent (default)
  - When copies being updated, can be returned an inconsistent (not yet updated) copy
  - Reads are fast, but not guaranteed consistent
  - All copies of data eventually become consistent within around a second
- Strongly consistent
  - When copies being updated, attempt to read will wait until copies are consistent before returning
  - Consistency guaranteed
  - Slower reads
  - All copies of data will be consistent within a second

Data stored on SSD storage and spread over 3 different AZs.  
Partitions:

- Allocation of storage for a table, backed by SSDs and automatically replicated across multiple AZs within a region
- Slicing a table up into smaller chunks of data (partitions) to speed up reads by logically grouping similar data.
- DynamoDB automatically creates partitions as data grows
  - Starts off with single partition
  - Creates new partitions:
    - For every 10GB of data
    - When exceeding maxes of read capacity units (RCUs) and 1000 write capacity units (WCUs) per partition
      - RCUs and WCUs evenly split amongst partitions

Primary Keys:

- Must be defined for every table
  - Cannot be changed later
- Determine where and how data will be stored in partitions
- Should be:
  - Distinct \- as unique as possible
  - Uniform \- divide data as evenly as possible
- Partition key (PK) determines which partition data should be written into
  - Using just a partition key is called a **simple** primary key
    - Must be unique
    - DynamoDBs secret internal hash function takes key and determines partition
- Sort key (SK) determines how data should be sorted on a partition
  - Using both partition and sort keys is called a **composite** primary key
    - Combination must be unique
    - Same hash function
    - Records stored together sorted according to sort key
- DynamoDB does not have a Date datatype
  - Need to use string for dates

Queries vs. Scans:

- Queries
  - Find items in a table based on primary key values
  - Query any table or secondary index with a composite primary key
  - Eventually consistent by default
  - Returns all attributes for items by default
  - Sorted ascending by default
- Scans
  - Look through all items, return filtered
  - Returns all attributes for items by default
  - Scan any table or secondary index
  - Operations are sequential
    - Speed up a scan using segments
  - Much less efficient than querying, especially at scale

## Amazon Keyspaces

Fully-managed Apache Cassandra database.

- Cassandra \= open source NoSQL key/value database like DynamoDB
  - Columnar store db with additional functionality
- Cluster \= collection of nodes
- Node \= holds 2-4 TB of data
  - All nodes read and write
  - Represent smallest unit of db
  - Data replicated on multiple nodes
- Ring \= node arrangement where all nodes connect to each other
- Keyspace \= namespace specifying data replication on nodes
- Table \= tabular data of columns and rows with a primary key
- Queried using Cassandra Query Language (CQL)
  - Similar to SQL
- Typically interact with Cassandra via an SDK
- Keyspaces allows from AWS Management Console:
  - Creation of keyspaces
  - Creation of tables
  - CQL queries (CQL editor)

## Amazon Neptune

Highly available and durable graph database with related offerings \- encompasses:

- Amazon Neptune Database
  - Two types:
    - Neptune Provisioned \- choose an instance type
    - Neptune Serverless \- set min/max Neptune Capacity Units (NCUs)
  - Supports multi-AZ deployments
  - Two storage configs:
    - Standard \- 25% cheaper
    - I/O optimised \- additional cost for input output optimisation
  - Can create Jupyter Notebook (within Amazon SageMaker Notebook) including magic extensions to easily work with Neptune database
  - Bulk Loader can be used to import large amounts of data
  - Can use Gremlin, SPARQL or OpenCypher
    - Gremlin \= graph traversal language
      - OLTP or OLAP queries essentially
      - Write once run anywhere (WORA)
      - Can use multiple languages to write Gremlin
      - Generally more proficient at traversal
    - OpenCypher
      - Easier to use than Gremlin
    - SPARQL \= Resource Description Framework (RDF) query language
      - Write queries against loosely key value data following RDF specification of W3C
  - Multiple built-in or third-party options for visualising the graph db
- Amazon Neptune Analytics
  - Extend the db using capabilities to run large-scale graph analytic algorithms efficiently on operation graph datasets
  - Load into Analytics from data in an S3 bucket or Neptune db
  - Integrated with Neptune Workbench
  - Includes algorithms like PageRank, shortest path, community detection directly on applicable data
  - Leverages Apache Spark cluster to perform complex analytics on graph data
- Amazon Neptune ML
  - Use graph neural networks (GNNs) (machine learning technique purpose-built for graphs) to make easy, fast and accurate predictions using graph data
  - Powered by Deep Graph Library (DGL)
  - Neptune integrates with LangChang

### Graph Database

Database composed of data structure using vertices (nodes) forming relationships between them (edges, arcs, lines). Nodes contain data properties, edges can contain relational data.  
Use cases:

- Fraud detection
- Real-time recommendation engines
- Social media graphing
- Machine learning / AI
- RAG

### Database Migration Service (DMS)

Quickly and securely migrate one database to another, including from on-premises database to AWS. Many source and target types available such as MySQL, PostgreSQL, MongoDB and many more.  
AWS DMS contains replication instances. The flow goes from the source database, into DMS, into the replication instance containing the replication task via a source endpoint, out to the target database via the target endpoint.  
AWS schema conversion used to automatically convert the source database schema to the target database schema. For data warehouses, the desktop app AWS Schema Conversion tool can be used (Windows and Linux only \- no Mac).  
Migration methods:

- Homogenous data migration
  - Migrate data with native database tools
  - Create migration project in DMS then performed using a serverless compute
  - Pay as you go model
- Instance replication
  - Provision an instance with chosen instance type to perform replications between databases
- Serverless replication (DMS serverless)
  - Pay as you go
  - No public IPs
  - Must use VPC endpoints to access specific AWS services (e.g. S3, Kineses, DynamoDB)
  - Limited selection of sources and targets
  - Does not support views with selection and transformation rules
