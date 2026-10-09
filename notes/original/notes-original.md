# NOTES: AWS Certified Solutions Architect \- Associate

# S3

## Buckets

Buckets are uniquely named per **partition** \- so effectively globally unique.  
No limit on number of files per bucket.  
Globally available but region specified on bucket creation

Names:

- Must be 3-63 characters
- Must begin and end with a letter or number
- Can only consist of lowercase letters, numbers, dots and hyphens
- Can’t contain double dots (..)
- Can’t be formatted like an IP address
- Can’t start with xn--, sthree- or sthree-configurator
- Can’t end \-s3alias or \--ol-s3
- Can’t contain dots if used in S3 transfer acceleration

Limit of 100 buckets \- can apply to increase to 1000  
Files can be **0b-5tb**  
2 types: **general purpose** buckets & **directory** buckets (S3 express one zone storage class)  
Standard S3 folders are just s3 objects with folder type, they do **not** contain any objects, objects are just renamed to inherit it as a prefix.

## Metadata

**ETags** are hashes of object contents to easily detect whether an object has changed  
**Checksums** ensure data integrity on upload or download by verifying amount of data in the file  
**S3 object metadata** can be system-defined or user-defined. **Not** the same as resource or objects tags.  
Usually **can’t** change system metadata (amazon control) but some you can (like **content type**)  
User metadata names must start **x-amz-meta-**

## Locking

**S3 object lock** prevents objects from being deleted or overwritten, following **WORM** (write once read many)  
Object locks can be **retention periods** (limited time) or **legal holds** (until further notice)  
Object locked S3 buckets **cannot** be destination buckets for server access logs  
Object locks are either **Governance** mode (privileged users can bypass) or **Compliance** mode (no one can bypass)  
Individual object level locks can be applied **only** via the AWS API.  
**Virtual Hosted-style** (DELETE /image.jpg, Host: examplebucket.s3.blahblah) and **Path-style** (DELETE examplebucket/image.jpg, Host: s3.blahblah) REST API requests are supported  
Can use **standard** endpoints (e.g. https\://s3.us-east-2.amazonaws.com) and **dualstack** endpoints (e.g. https\://s3.**dualstack**.us-east-2.amazonaws.com). Dualstack handles **IPv4 & IPv6**, standard only **IPv4**.

## S3 Storage Classes (per file)

| Name                                                       | Description                                                                                                                                  | Costs                                                                   |
| ---------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| **Standard**                                               | \-                                                                                                                                           | Per storage usage and request                                           |
| Reduced Redundancy Storage (**RRS**)                       | **Legacy** class. Can still select for now but will be retired.                                                                              | Comparable to standard                                                  |
| **Intelligent Tiering**                                    | AI analyses object usage and determines storage class                                                                                        | Extra fee to analyse                                                    |
| **Express One-Zone**                                       | Single-digit ms access performance. Only one user-selected AZ (availability zone). Stored in Directory bucket.                               | 50% less than standard                                                  |
| **Standard IA** (Infrequent Access)                        | Fast, cheaper if you need to access a file less than once per month. Minimum **30 days** storage                                             | Extra fee to access. 50% less than standard.                            |
| **One-Zone IA**                                            | Standard IA but in one user-selected AZ. Not as quick access as express one-zone. Minimum **30 days**.                                       | Cheaper than standard IA by 20%                                         |
| Glacier **Instant** Retrieval                              | Long term data storage, get data instantly. This one actually uses S3. **90 days** minimum                                                   | About 6x cheaper to store data but access costs. 68% less than standard |
| Glacier **Flexible** Retrieval (Formally S3 Glacier \*ish) | Get data in minutes-hours. Uses S3 and vaults under the hood. **40KB** added metadata per file. 3 speeds of retrieval (1-5mins, 3-5h, 5-12h) | Cheaper than instant                                                    |
| Glacier **Deep Archive**                                   | Same as flexible retrieval but lower storage costs and higher retrieval costs. 2 retrieval modes, standard and bulk (both 12-48h)            | Cheapest                                                                |
| **Outpost**                                                | AWS gives you the servers to host yourself                                                                                                   | Flat upfront cost not usage-based                                       |

![][image1]

## Security

**Bucket Policies** define permissions for an S3 bucket using JSON-based access policy language

- Provide access to single bucket and its objects
- Overlap with IAM policies but bucket policies are per bucket and can specify multiple principles
- Block public access will take priority over bucket policies if on

**Access Control Lists (ACLs)** are a legacy method of defining access permissions to objects and buckets

- Allow access to other accounts, not users in your account
- No conditional permissions or denying permissions

**AWS PrivateLink** enables private network access to S3, bypassing public network for enhanced security

- Instance of internetwork traffic privacy
- Also referred to as VPC Interface Endpoints
- Allows to connect Elastic Network Interfaces (ENIs) directly to other AWS services
- Can also connect to select third-party services via AWS Marketplace
- Can go cross-account
- Fine-tuned permissions via VPC endpoint policies
- Costs to use

**VPC Gateway Endpoints** connect a VPC directly to S3 or DynamoDB , staying private within the AWS network

- Another instance of internetwork traffic privacy
- Cannot go cross-account
- Does not have fine-tuned permissions
- Free to use

**Cross-Origin Resource Sharing (CORS)** allows restricted resources on other domain web page to be requested

- CORS configuration on static website-hosting S3 buckets can allow different origins to perform HTTP requests through the static website
- Config can be in JSON or XML (JSON encouraged \- console is json only)

**Block Public Access** offers access blocks to the public on S3 resources

- Enabled by default
- 4 options: new ACLs, any ACLs, new Bucket Policies / Access Points, any Bucket Policies / Access Points

**IAM Access Analyzer** analyses resource policies to identify areas of potential risk

- Need to create analyzer at the account level

**Internetwork Traffic Privacy** encrypts data travelling between AWS services and the internet  
**Object Ownership** manages data ownership between AWS accounts when objects are uploaded to buckets  
**Access Points** simplify managing data access at scale for large shared datasets

- Named network endpoints attached to buckets
- Perform object operations only
- Distinct permissions via Access Point Policies
- Distinct network controls: access via internet or specified VPC
- Instead of a big Bucket Policy, you could have multiple Access Points with their own policies (e.g. internet access for public files, then dev and prod access points)
- **Multi-Region Access Points** involve using one global endpoint to access buckets in multiple regions in order to route to the one with the lowest latency
  - Good S3 replication rules help here for data consistency.
  - Uses AWS Global Accelerator under the hood
- **Object Lambda Access Points** allow you to transform the output requests of S3 objects
  - Example use case: scrubbing/redacting sensitive information from a file before presenting it
  - Original objects remain unmodified
  - Can be performed on HEAD (info on an object), GET (the object itself) and LIST (list of objects) requests.
  - Lambda function is attached to a bucket via Object Lambda (multiple can be attached)
  - Request goes through the access point to get to the object lambda

**Access Grants** provide access to data via directory services e.g. Active Directory  
**Versioning** allows recovery and restoration of old versions of objects  
**MFA Delete** requires multi-factor authentication for deletion of objects  
**Object Tags** allow for categorisation of objects using key-value pairs  
**In-Transit Encryption** protects data as it travels to and from S3 services over the internet

- Only encrypts for the transfer process, not during storage
- Uses TLS (Transport Layer Security) \[at least v1.2\] or SSL (Secure Sockets Layer) \[deprecated\]

**Server-Side Encryption** encrypts data as it is written into S3 and decrypts on download

- On by default
- **SSE-S3**: S3 handles keys, encrypted using AES-GCM (256)
  - Default option
  - Keys rotated automatically
  - Each object encrypted with unique key
  - No additional charge
- **SSE-KMS**: Keys managed via AWS KMS (Key Management Service)
  - Create managed key in KMS then choose it to encrypt object
  - KMS automatically rotates keys
  - Key policy controls who can decrypt using key
  - Good for meeting regulatory compliance standards
  - Not cross-region \- keys must be same region as objects
  - kms:GenerateDataKey and kms:Decrypt needed respectively
  - Small additional charge per key & request
- **SSE-C**: You handle keys yourself and upload
  - No additional charge
  - Provide encryption key with every request
  - S3 stores a salted hash of encryption key to validate future requests
  - Beware: different object versions can be encrypted with different keys
- **DDSE-KMS**: Dual-layer encryption
  - Data encrypted client side and server side
  - The key for client-side encryption comes from KMS
  - No additional charge
  - Client requests KMS to generate data encryption key (DEK) using a customer-managed key (CMK), KMS sends plaintext version and encrypted version of DEK, plaintext one used to encrypt data client-side, encrypted version uploaded and stored with encrypted data in S3
  - Client retrieves encrypted data and encrypted DEK, client sends encrypted DEK to KMS which sends back plaintext DEK, plaintext DEK used to decrypt data
  - Plaintext DEK not kept
- Content is encrypted \- metadata is not

**Client-Side Encryption** encrypts data before uploading to S3 and decrypts once downloaded

- **S3 Bucket Key** allows you to generate short-lived bucket-level keys, stored in S3
  - Reduces request volume costs and overall traffic
  - Generates a unique key per requester
  - Can enable for all new objects in a bucket or at object level
  - Can be enabled for SSE-S3 and SSE-KMS

**Compliance Validation** ensures S3 services meet requirements for compliance standards (GDPR etc.)  
**Infrastructure Security** protects the underlying infrastructure of the S3 services, ensure data reliability and availability

## Data

**Data Consistency** is whether or not data being stored in two different places is exactly the same or not.  
**Strongly consistent** is when every request can expect consistent data to be returned \- the data is ensured to be consistent before returning.  
**Eventually consistent** is when out of date (inconsistent) data may be returned at first and corrected when consistency is established.  
S3 has strong consistency for all read, write and delete operations.

**Object replication** is often used to duplicate objects while maintaining metadata, duplicate objects into different storage classes or under different ownership, store objects over multiple regions and more.  
4 replication options: **Cross-Region** (live), **Same-Region** (live), **Bi-Directional** (live) or **Batch** (on demand).

**Versioning** allows the storage of multiple versions of objects at the same object key address. It is **disabled by default** on buckets and when turned on, it **cannot** be turned off again \- just suspended.

## Tools

**S3 Lifecycle** allows you to automate the transition and expiration of objects using set rules, and can apply to both current and non-current versions of objects. Filters can be applied in rules to only affect qualifying objects.

**S3 Transfer Acceleration** is a bucket-level feature providing fast and secure transfer of objects over long distances between the bucket and end users. It uses **CloudFront**’s distributed Edge Locations to quickly enter the **Amazon Global Network**. Instead of going straight to the bucket address, users navigate to an edge location using _s3-accelerate(.dualstack)_ host (only works on virtual-hosted style requests). It can take 20 minutes after Transfer Acceleration is enabled to become active.

**Presigned URLs** provide temporary access to upload or download objects via URL.

**Mountpoint** allows you to mount an S3 bucket to a Linux file system. It **can** read files up to 5TB, list files, read files and create new files. It **cannot** modify existing files, delete files or support symbolic links or file locking. Ideal for apps that don’t need all the features of a shared file system but require S3’s elastic throughput to read and write large datasets. Available in Standard, IA, RRS and Glacier Instant storage classes.

**Archived objects** are ones which cannot be accessed in real-time, in exchange for lower storage costs. Archive storage classes require manual data transfer and are best for when you know your access patterns as you get the lowest storage costs (Glacier storage classes). Archive access tiers automatically move data based on usage and so are best for when you don’t know your usage patterns and this costs slightly more (Intelligent Tiering).

**Requesters Pay** is an option to offload payment incurred for requests and downloads to the one making the request. Storage costs are not offloaded. All requesters must authenticate, anonymous access is disabled. **403** (Forbidden Request) responses will be received if the request fails to authenticate properly or the request is using SOAP. No charge on 403 responses.

**AWS Marketplace** provides alternatives to AWS services which work with AWS.

**S3 Batch Operations** perform large scale operations on S3 objects in batch. Can do: copy, invoke lambda function, replace object tags, replace ACLs, restore, object lock retention or object lock legal hold. Need to supply list of S3 objects or an Inventory Report manifest in json. Can generate Completion Reports (through CLI).

**S3 Inventory** takes inventory of all objects in a bucket on a repeating schedule for an audit history of changes (output into another S3 bucket). Can be **daily** (delivered within 48h), or **weekly** (delivered first within 48h, then every Sunday). Outputs are **CSV, ORC** or **Parquet**. Metadata can be included in the report. Can filter on prefixes or to current versions of objects.

**S3 Select** lets you use SQL to filter objects **based on content**. Works on objects stored in CSV, JSON or Parquet. Output can be CSV or JSON. Works on CSV or JSON files with GZIP or BZIP2 compression. Works on **server-side** encrypted objects. Can be used in Standard, Intelligent Tiering, IA and Glacier Instant Retrieval storage classes.

**S3 Event Notifications** allows a bucket to notify other AWS services about changes/events \- making application integration easy for S3. Notification enabled events: **object creation, object removal, object restored, RRS object lost, replication, lifecycle expiration, lifecycle transition, intelligent-tiering automatic archival, object tagging** and **object ACL PUT.** Event notifications can go to Simple Notification Service (**SNS**), Simple Queue Service (**SQS**), **Lambda** function or **EventBridge**. Notifications designed to be **delivered at least once**. Notifications take seconds to a minute or longer.

**Storage Class Analysis** is a way of analysing access patterns of objects within a bucket to recommend objects to move between **standard** and **IA**. Can use up to **1000** filters and can export to **CSV**. More manual and smaller-scope but more cost-effective than intelligent-tiering.

**Storage Lens** is a storage analysis tool for the entire AWS organisation to analyse total storage, trends etc. from a dashboard (updated daily). Usage and metrics can be exported to **CloudWatch** and metrics can be exported as **CSV** or **Parquet** to a bucket.

**Static Website Hosting** allows you to host and serve **static** websites from your S3 bucket. No server-side interactivity, but client-side is possible. Does **not** support **HTTPS** (would have to use **CloudFront**). Will be hosted at http\://bucket-name.s3-website**\-**region.amazonaws.com or http\://bucket-name.s3-website**.**region.amazonaws.com (dot or dash before region). Cannot use requesters pay. Two hosting types: host static website, redirect requests to objects.

**Multipart Upload** allows single object uploads to be split into multiple parts which can be uploaded at any time (no expiry). You can upload files while you’re writing them. Recommended for files larger than **100MB**. Send multipart upload request, get back upload ID to send with future parts before a final ‘upload complete’ alert. Can upload parts concurrently. It’s also possible to download just specified **byte ranges** of objects (this can also be done concurrently). This could be useful to download large files in parts and piece back together.

**Interoperability** in cloud services is the ability for services to exchange and utilise information seamlessly with one another. Many services output exports or logs to S3.

Can set S3-specific settings in the config file (\~/.aws/config) e.g. max queue size, max bandwidth etc.

# AWS API

Management Console, CLI, SDKs and plain HTTP requests all access resources via the AWS API (Application Programming Interface).

## CLI

**Terminal** \= text only interface  
**Console** \= physical computer to input information into a terminal  
**Shell** \= command line program user interact with to input commands

AWS **CLI** is a **Python** executable program (Python is required)  
Can be installed on Windows, Mac or Linux.

Order of priority for variables \= CLI **parameters** \> **env** vars \< **config** files.

## Keys

**Access keys** are generated **per user** (can have **two** active keys). These keys are used to authenticate and have the same access as the user. Two methods of using keys: storing in **\~/.aws/credentials** (TOML format), which can provide keys for multiple profiles, and **exporting** as env vars.

## Retries

Common to get networking issues when interacting with any API over a network. Standard is to **retry** with **exponential backoff** (e.g. retry in 1 second, retry in 2 seconds, retry in 4 seconds etc.). Often this is built into SDKs or CLIs, with options.

## STS

Security Token Service (**STS**) is a way of getting temporary, limited-privilege credentials for IAM **users** or federated users. Global service, all requests go to single endpoint **http\://sts.amazonaws.com**. Will return: **AccessKeyID, SecretAccessKey, SessionToken** & **Expiration**. Could use instead of long term access keys for more security by having user with permissions to assume roles but not access directly and use STS every time.

## Signing

Requests to AWS usually need to be **signed** to prevent data tampering and verify requester identity (not for anonymous access to S3 or select other requests). Requests using an SDK or CLI are signed automatically. **2 protocols** for signing: Signature **v2** (legacy) and Signature **v4**.  
In signature v4, select request elements are concatenated into a **‘string to sign’**, a **signing key** is created from the **secret access key** , a **signature** is created from a hash-based message authentication code (**HMAC**) of the **string to sign** using the **signing key**.  
The process to sign depends on the request type: usually in authorisation header, but can be in a post request or query parameters.

## IP Address Ranges

AWS publishes all IP ranges in JSON at **https\://ip-ranges.amazonaws.com/ip-ranges.json**. This can be useful for whitelisting IP addresses for example.

## Service Endpoints

To connect programmatically to an AWS service, you use the URL of the entry point to the service (**endpoint**) in the format: **protocol://service-code.region-code.amazonaws.com**. **Four** types: **Global**, **Regional** (region must be specified), **FIPS** (enterprise use, uses encryption), **Dualstack**. These types can be combined. A service can have **multiple** endpoints. SDKs and the CLI automatically use the default endpoint for a service in a region.

# VPC

## Definition & Key Features

AWS Virtual Private Cloud (**VPC**) is a **logically isolated virtual network**, resembling a traditional network you’d operate in your own datacentre. Virtual Machines (e.g. EC2) are the most common reason for using a VPC in AWS. Other most common is Virtual Network Cards (e.g. ENIs) which are used within a VPC to attach to different compute types (e.g. EC2, Lambda, ECS). VPC tightly coupled with EC2, VPC **CLI commands are under aws ec2**.

VPCs are **region-specific**, they do **not** span cross-region (can use VPC peering to connect across regions).  
Up to **5** VPCs **per region** (adjustable with request).  
Up to **200** subnets **per VPC**.  
Up to **5** IPv4 and **5** IPv6 CIDR blocks **per VPC** (both adjustable to 50 with request).  
Every **region** comes with **default** VPC (it **can** be deleted but also recreated). Comes with

- **IPv4 CIDR** block at **172.31.0.0/16**
- A subnet for **each** possible AZ
- An **IGW**
- A default **SG**
- A default **NACL**
- Default **DHCP** options set
- A **route table** with route out via **IGW**

Do **not** cost money: VPCs, route tables, NACLs, internet gateways, security groups, subnets, VPC peering (itself)  
**Do** cost money: NAT gateways, VPC endpoints, VPN gateways, Customer gateways, IPv4 addresses, Elastic IPs.  
**DNS hostnames** if your instance has them.

## Core Components

Internet Gateways (**IGW**s) connect a VPC to the internet (can be egress-only).  
**Virtual Private Gateways** (VPN Gateways) connect a VPC to a private external network.  
**Route Tables** determine where to route traffic within a VPC.  
**NAT Gateways** allow private instances (e.g. virtual machines) to connect to services **outside** the VPC (not needed for IPv6 as all addresses are public, just IPv4).  
Network Access Control Lists (**NACL**s) act as **stateless** virtual firewalls for compute within a VPC and operate at the **subnet level** with allow **and deny** rules.  
Security Groups (**SG**) act as **stateful** virtual firewalls for compute within a VPC and operate at the **instance level** with allow rules.  
**Public Subnets** allow instances to have public IP addresses.  
**Private subjects** **dis**allow instances from having public IP addresses.  
**VPC Endpoints** connect privately to AWS support services.  
**VPC Peering** connects VPCs to other VPCs.

VPCs contain subnets, which can contain EC2 instances.

## Deleting

To **delete** a VPC, you must first delete:

- SGs
- NACLs
- Subnets
- Route Tables (RTs)
- Gateway endpoints
- IGWs
- Egress-only internet gateways (EO-IGWs)

When you delete via the management console, it will **automatically** **attempt** to delete these for you.

## Default Route (Catch-All Route)

The **default** route or **catch-all** **route** represents all possible IP addresses. This is essentially giving access from **anywhere** or to the internet **without restriction**.  
Two versions: IPv4 (**0.0.0.0/0**) and IPv6 (**::/0**)  
IPv4 is now charged, to encourage IPv6 use.

## Shared VPCs

VPCs can be shared with other AWS **accounts** within the **same organisation** using AWS Resource Access Manager (**RAM**).  
This really just shares **subnets**. Can**not** share **default** subnets.  
You must **enable** sharing within the organisation via the RAM **API**.  
Must create: a **resource share** (what is being shared) and **shared principles** (who is being shared with) in **RAM**.  
Sharing does **not** necessarily mean **visibility** of resources which have been shared with you, **cross-account roles** are required.

## NACLs

Network Access Controls (**NACLs**) are **stateless** firewalls at the **subnet level** which have both allow **and deny** rules.  
A **default** NACL is created with **every** VPC (do not span VPCs).  
NACLs contain **2** sets of rules: Ingress/**Inbound** rules and Egress/**Outbound** rules.  
Subnets are associated with **exactly one** NACL.  
Rules have **numbers** (priority **lowest** **\>** **highest** number).  
Recommendation is to increase in 10s or 100s.

## Security Groups (SGs)

**Stateful** virtual firewalls at the **instance level** (e.g. EC2 instance). Inbound **and** Outbound rules, but only allow, **no block**. An SG can be associated with **multiple** instances in **different** **subnets**, but they must be in the **same VPC**.  
Can:

- Allow IP addresses
- Allow another security group
- Nest security groups

Limits:

- 10k SGs per region (default is 2.5k)
- 60 inbound **and** 60 outbound rules per SG
- 16 SGs per ENI (default is 5\)

Do not filter traffic from:

- Amazon Domain Name Services (DNS)
- Amazon Dynamic Host Configuration Protocol (DHCP)
- Amazon EC2 instance metadata
- Amazon EC2 task metadata endpoints
- License activation for Windows instances
- Amazon Time Sync Service
- Reserved IP addresses used by the default VPC route

## Stateful vs Stateless

**Stateless** firewalls like **NACLs** are **not** aware of the state of the request. They will stop traffic in **both directions** to check it is allowed through.  
**Stateful** firewalls like **SGs** **are** aware of request state. They will only stop to check **inbound requests** \- outbound traffic is automatically allowed through and as is inbound responses to outbound requests.

## Route Tables

**Route Tables** (RTs) contain a set of routes and are used to determine where network traffic is directed. Each **subnet** in a VPC must be associated with a route table (implicitly or explicitly). A subnet can only be associated with **one** RT at a time, but **multiple subnets** can be associated with the **same RT**.  
There is a **default** route table created with every VPC which **cannot** be deleted. If a subnet is not explicitly associated with a route table, it will use the default (Main) route table.  
Routes use **destinations** (final intended point) and **targets** (next step towards destination) \- e.g. Destination=172.31.0.0/16, Target=local.  
Targets:

- Local \- default local route allowing associate subnets within VPC route to the other route (**cannot** delete)
- IGW \- ingress **&** egress connections to internet (IPv4 & IPv6)
- Virtual Private Gateway (VPG) \- out to private connection to on-premise network
- NAT Gateway \- egress connections for private instances out to internet (IPv4)
- Egress Only Internet Gateway (EO-IGW) \- egress connections for private instances out to the internet (IPv6)
- Instance \- out to specific EC2 instance
- Network Interface (ENI) \- out to specific Elastic Network Interface
- Carrier Gateway \- out to managed Wide Are Network (WAN) via AWS Cloud WAN
- Gateway Load Balancer Endpoint (GWLB) \- out to GWLB (used for third-party virtual applications)
- Outposts Local Gateway \- out to physical server rack with AWS services in own datacenter
- Peering Connection \- out to another VPC
- Transit Gateway (TGW) \- out to a transit hub for connecting multiple VPCs and on-premises network

## Gateways

A **Gateway** is a networking service which sits between two different networks. They often act as reverse proxies, firewalls and load balancers.  
Types:

- Internet Gateway (**IGW**) \- Inbound & outbound public traffic for IPv4 & IPv6
  - Needs to be a **target** in your route table even when associated with a VPC
  - Network Address Translation (**NAT**) is required for instances assigned public IPv4 addresses (**not** IPv6 as all public)
  - **Default** VPCs come with IGW, for others must be created and assigned
- Egress-Only Internet Gateway (**EO-IGW**) \- Outbound private traffic for IPv**6**
  - Allow outbound traffic to internet, prevent inbound traffic
- **NAT** Gateway \- Outbound private traffic for IPv**4**
- Carrier Gateway \- Connect to AWS partnered telecom network
- Virtual Private Gateway \- Endpoint into AWS account for a VPN connection
- Customer Gateway \- Endpoint into on-premise account for a VPN connection
- Gateway Load Balancer (GWLB) \- Network layer (layer 3\) load balancer to run and scale third-party virtual applications e.g. firewalls
- Direct Connect Gateway \- Endpoint to a fiber optic connection at co-location data center
- AWS Backup Gateway \- Endpoint for AWS managed backups
- IoT Device Gateway \- Endpoint to send IoT data both directions
- AWS Transit Gateway \- Hub and spoke model to simplify VPC peering
- Amazon API Gateway \- Abstracts API endpoints to services
- AWS Storage Gateway \- Syncing, caching or extending local storage to cloud storage

Gateways are sometimes called **Load Balancers** or vice versa.

## IPs

**Elastic IP** (EIP) addresses are static IPv**4** addresses which **do not change**. When you restart an EC2 instance for example, its IP address will be **different** upon restart which could break things. EIPs remain the same and can be **remapped** to different instances or network cards (**ENIs**).  
Elastic IPs are:

- Region-specific
- Drawn from Amazon’s pool of IPv4 addresses (and include the public IPv4 charge)
- Charged at \$1 for each unassociated address to encourage releasing back to the pool when unused

You can:

- Set EIPs to automatically reassociate with the same instance or network interface in the case of failure or restart (can also explicitly set to not reassociate although this is default)
- Attempt to recover a specific address
- Bring a custom address (have to import it first)

**All** AWS service support IPv4 \- **not** all services/resources have IPv6 turned on by default, but they will support it  
IPv**4**\-only VPCs can be migrated to dual stack (cannot disable IPv4 support for VPC/subnets):

1. Add new IPv6 CIDR block to VPC
2. Create or associate IPv6 subnets (IPv4 subnets cannot be migrated)
3. Update route table to IPv6 to IGW
4. Upgrade SG rules to include IPv6 address ranges
5. Migrate EC2 instance type if it doesn’t support IPv6

## AWS Direct Connect

**Direct Connect** establishes dedicated network connections from on-premises locations to AWS for very fast, reliable and secure connections for enterprises.  
**Two** bandwidth options: **Lower** (50MB-500MB/s), **Higher** (1GB, 10GB, 100GB/s)  
IPv4 **and** IPv6 support.  
Reduces network costs and increased bandwidth throughput but considerable cost overall.  
**Requirements**:

- Co-located with existing direct connect location
- Working with direct connect partner who is member of AWS Partner Network (APN)
- Working with independent service provider to connect to direct connect
- Network requirements (which aren’t important to know).

Factors to **pricing**:

- Capacity (port size) \- max rate of data transfer possible
  - Larger port size \= higher cost
- Port hours \- time a direct connect port is provisioned for use (regardless of data transfer)
  - Dedicated: physical connection between your network port and AWS network port (billed /h via AWS)
  - Hosted: logical connections between AWS partner and AWS network port (billing subject to partner)
- Data transfer out (DTO) \- outbound traffic sent through direct connect to destinations outside AWS
  - Data transfer **in** is free
  - Data transfer **within same region** is free

## VPC Endpoints

VPC Endpoints allow private connection between a VPC and other AWS services, **without** having to leave the AWS network and use the internet.  
Benefits:

- Eliminates the need for:
  - IGW
  - NAT
  - VPN
  - Direct Connect
- Instances in the VPC do **not** require a **public** IPv4 address
- Higher security without adding availability risks or bandwidth constraints on traffic
- **Horizontally** scaled, redundant and highly available VPC component

3 types:

- **Interface endpoints**
  - Use **PrivateLink**
  - ENIs with a private IP address that serve as entry point for traffic going to supported service
  - **Cost** money \- per hour & data processed
  - **B**idirectional
- **Gateway endpoints**
  - **Not** PrivateLink
  - Connects to only **S3** and **DynamoDB**
  - Specify the VPC to contain the endpoint and the desired service
  - **Un**idirectional
- **Gateway Load Balancer endpoints**
  - Use **PrivateLink**
  - Type of **Interface Endpoint**
  - **Costs** money \- per hour & data processed
  - **Uni**directional (usually)
  - **ENIs** allowing traffic distribution to **multiple** network virtual appliances \- commonly **security**\-related:
    - Firewalls
    - Intrusion Detection and Prevention Systems (IDS/IPS)
    - Deep Packet Inspection Systems
  - Incoming traffic hits GWLB **endpoint** and goes over PrivateLink to **GWLB** which distributes to appliances, then back via PrivateLink into the ENI and onto the application
  - Send traffic to **GWLB** by making config changes in VPC’s **route tables**
  - Can get virtual appliances as a service from:
    - AWS Partner Network (APN)
    - AWS Marketplace

## Private Link

**AWS PrivateLink** is a service allowing secure connection of a **VPC** to supported AWS services, services hosted in other AWS accounts and supported AWS Marketplace partner services, **without** needing an IGW, NAT, NPM or Direct Connect connection. PrivateLink ready partner services allow you to access SaaS products privately as if they were running in your VPC.  
An **Interface Endpoint** connects the VPC to PrivateLink, which then connects to the **Service Endpoint** of the AWS service / service provider VPC.

## VPC Flow Logs

**Flow Logs** capture IP traffic information running through a VPC when turned on.  
Can be scoped for:

- VPC, subnets
- ENIs
- Transit Gateway
- Transit Gateway Attachment

Can monitor traffic for:

- **ACCEPT**(ed traffic)
- **REJECT**(ed traffic)
- **ALL** (traffic)

Logs delivered to any of:

- **S3** bucket
- **CloudWatch** Logs
- **Kinesis** Data Firehouse

Format of logs:

1. version \- VPC Flow Logs version
2. account-id \- AWS account ID for flow log
3. interface-id \- ID of network interface for which traffic recorded
4. srcaddr \- Source IPv4 (private) or IPv6 address of network interface
5. dstaddr \- Destination IPv4 (private) or IPv6 address of network interface
6. srcport \- Traffic source port
7. dstport \- Traffic destination port
8. protocol \- IANA protocol number (Assigned Internet Protocol Number) of traffic
9. packets \- Number of packets transferred during capture window
10. bytes \- Number of bytes transferred during capture window
11. start \- Time (Unix seconds) of capture window start
12. end \- Time (Unix seconds) of capture window end
13. action \- Action associated with traffic (**ACCEPT**/**REJECT**)
14. log-status \- Flow log logging status
    1. **OK** \- Data logging normally to chosen destinations
    2. **NODATA** \- No network traffic to or from network interface during capture window
    3. **SKIPDATA** \- Some flow log records skipped during capture window (could be internal capacity constraint or internal error)

## AWS VPN

**AWS Virtual Private Network** establishes a secure and private tunnel from a network or device to the AWS global network.  
**Site-to-Site** VPN:

- Securely connects on-premises network or branch office site to VPC
- Main components:
  - **VPN connection** \- secure connection between VPC and on-premises equipment (logical abstraction)
  - **VPN tunnel** \- encrypted connection for data (specific pathway making up connection \- AWS usually provides **2 tunnels** for site-to-site for availability)
  - **Customer Gateway** (CGW) \- provides info to AWS about customer gateway **device**
    - Presents: BPG ASN for customer gateway device, IP address of customer gateway external device and private certificate provisioned by AWS Certificate Manager (ACM)
    - Also need to provide additional config
  - **Customer Gateway Device** \- Physical device or application on **customer side** of VPN connection
  - **Target Gateway** \- VPN endpoint on **Amazon side** of VPN connection (logical abstraction)
  - **Virtual Private Gateway** (VGW) \- VPN endpoint on **Amazon side** of VPN connection, attached to single VPC (specific resource)
    - When creating VGW, must assign Amazon Autonomous System Number (**ASN**) or custom ASN (default ASN=64512)
    - ASN is unique identifier globally allocated to each autonomous system participating in the internet
    - **Cannot** be changed once created
  - **Transit Gateway** \- Transit hub interconnecting **multiple** VPCs and on-premises networks, acting as a VPN endpoint for **Amazon side** of VPN connection
    - Can support IPv4 **and** IPv6
- Features:
  - **NAT** traversal
  - **CloudWatch** metrics
  - Reusable IP addresses for **customer** gateways
  - Additional **encryption** options
  - Support for IPv**6** traffic using a **transit gateway**\!
  - Can enable AWS **Global Accelerator** for connection
  - Can attach VPN to AWS **Transit Gateway**
- Limitations:
  - IPv**4** traffic **not** supported on **virtual private gateway** (must use **transit gateway**)
  - Does **not** support Path MTU Discovery
  - Recommend using **non**\-overlapping CIDR blocks for networks

Client VPN:

- Securely connects users to AWS or on-premise networks.
- Uses a **single** tunnel
- Uses **security groups** or **AD groups** for granular control
- User connects using open **VPN client** (AWS VPN Desktop Client is available in self-service portal)
- Has two roles:
  - **Admins** \- responsible for space setup and config
  - **Clients** \- who connect to client VPN endpoint to establish connection
- Can use:
  - **Certificate**\-based authentication (**Mutual** authentication)
  - **Active Directory** authentication (AWS **Directory Service**)
  - **Federation** authentication (**SSO SAML**)
- Can be used to securely connect to **RDS** instance in **private** **subnet**

Internet Protocol Security (**IPsec**) is a secure network protocol suite that **authenticates** and **encrypts** packets of data, providing secure encrypted communication between two computers over an **internet protocol** network \- used in **VPNs**.

## Network Address Translation (NAT)

**NAT** is a method of **mapping IP address** space into another by modifying network address info in the IP header of packets while in transit across a **traffic-routing** device.  
Uses:

- Re-mapping **private** IPv**4** addresses to **public** ones to access the internet from a private network
- Making **conflicting** network addresses of two networks more agreeable for unambiguous reference

**NAT Gateway**:

- Fully **managed NAT service** allowing instances in **private subnet** to establish **outbound** connections
- Required **per subnet** you want to outwardly connect
- **Costs** per hour and per GB data processed
- Two connection modes:
  - **Public** (default):
    - Connect instances in **private subnets** to the **internet**
    - **Cannot** receive unsolicited **inbound** connections from the internet
    - **Must** associate an **Elastic IP** (EIP) address (with additional cost)
  - **Private**:
    - Connect instances in **private subnets** to **other VPCs** or **on-premises network**
    - Can route traffic from NAT gateway through **transit gateway** or **virtual private gateway**
    - Can**not** associate **EIP** address
- Performs:
  - **DNS64** \- Translates **DNS requests** from IPv**4** to IPv**6** (like translating a phone book), once per connection
  - **NAT64** \- Translates **IP headers** of packets in transit in **both** directions between IPv**4** and IPv**6** to facilitate communication

**Nat Instances** are **legacy** deployments of NAT onto **individual EC2 instances** which have not been deprecated. These required the customer to handle scaling themselves. It may still be possible to use them via community AMIs to launch NAT instances to replace the deprecated Amazon one.

## Bastion / Jumpbox

**Bastions / Jumpboxes** (same thing, two names) are **security-hardened virtual machines** providing secure access into **private subnets** via SSH or RCP. NAT gateways should **not** be used as Bastions as they are only intended for **outbound** access to the internet. AWS does **not** have its own Bastions, third party / community ones are available. System Manager’s **Sessions Manager** can replace the need for a Bastion.

## VPC Lattice

**VPC Lattice** is a fully-managed **application networking service** which connects, secures and monitors services in an application. Easy way to turn resources into services for **micro-services** architecture.  
Can:

- Be used in **single** VPC
- Be used across multiple **VPCs**
- Be used across multiple **accounts**
- Perform **NAT** for IPv4, IPv6 (for translation) and overlapping networks
- Integrate with **IAM**
- Wight route for traffic (blue/green, canary deployments)
- Use **HTTPS** for service-service communication
- Use **custom** domain names
- Bring own SSL/TLS certs

How it works:

- **Service Network** is the logical container for all services which can communicate with one another in associated VPCs
- **Listeners** are the protocol and port the service listens to
  - Up to 2 listeners per service
  - Contains routing rules and a default rule
- **Target Groups** are collections of resources of a specific type
  - EC2, IP addresses, Lambda functions, ALBS, K8s pods
- **Service Directory** is a central registry of all VPC Lattice services owned by or shared with your account through AWS Resource Access Manager (**RAM**)
- Looks similar to load balancers but not the same

## Transit Gateway

**Transit Gateways** are network transit hubs that interconnect **VPCs** and **on-premise networks**.

- Leverage AWS Resource Manager (**RAM**)
- Operate as virtual **router** at **region level**
- Can attach **5000** VPCs to **each** gateway
- An **ENI** is provisioned in target VPCs to facilitate VPC to transit gateway communication (possible cost)
- Each attachment handles **50GB/s** of traffic
- Can use to:
  - Attach AWS **VPN** connections
  - Attach **Direct Connect** connections
  - Attach **third-party virtual appliances**
    - Via transit gateway attachments
    - Can be source **and** destination for packets (**bi**directional)
  - **Peer** connect with other transit gateways

## Traffic Mirroring

**Traffic Mirroring** sends copies of network traffic from a source ENI to a target ENI (or UDP-enabled NLB or GWLB). Often used to send traffic copies to a security monitoring appliance.

- Attaches VXLAN header
- Each packet mirrored once
- Filter rules can be applied (priority order)
- Create a mirror source and target

## Network Firewall

**AWS Network Firewall** is a **stateful**, managed network firewall for intrusion detection (**IDS**) and intrusion protection (**IPS**) for **VPCs**. Filters inbound **and** outbound traffic at **perimeter** of VPC. Uses open-source Suricata software under the hood.  
Use cases:

- Allow only traffic from known AWS service domains/IP address endpoints (e.g. S3) to pass
- Limit types of domain names apps can access using custom lists
- Deep packet inspection on traffic entering or leaving VPC
- Filter protocols (e.g. HTTPS)

## VPC Peering

**VPC Peering** connects one VPC with another over a direct network route using private IP addresses.

- **Not** a gateway or VPC connection
- Does **not** rely on separate physical hardware
- **No** single point of failure or bandwidth bottleneck
- Can be between IPv4 **or** IPv6 addresses
- Behave like on same network when connected
- **Can** span different AWS **accounts** and **regions**
- **Star** configuration \- 1 central VPC, 4 other VPCs
- **No** transitive peering \- must be direct (use transitive gateway for this)
- **No** overlapping CIDR blocks
- Data transfer **cross** **AZ** or **region** **costs**
- **Can** reference security groups in peered VPC in your security group rules

To VPC peer:

1. **Create** peering connection
2. **Accept** peering connection
3. **Add route** to route table for each VPC

## Network Address Usage

Network Address Usage (**NAU**) is a measure of resources in a virtual network.  
Limits:

- **VPC**s limited to **64**,000 NAUs (256,000 with an increase)
- **Peered VPCs** limited to **128**,000 NAUs (512,000 with increase) \- applies to **total** of peered VPCs

Can use **CloudWatch** NAU metrics to automatically keep track of NAUs (must enable NAU monitoring on VPC).

| Resource                                           | NAU Units |
| -------------------------------------------------- | --------- |
| Each IPv4/6 addressed assigned to ENI              | 1         |
| Each additional ENI attached to EC2 instance       | 1         |
| Prefix assigned to network interface               | 1         |
| NWLB per AZ                                        | 6         |
| VPC endpoint per AZ                                | 6         |
| Transit gateway attached                           | 6         |
| Lambda function attached                           | 6         |
| NAT gateway attached                               | 6         |
| Elastic File System (EFS) attached to EC2 instance | 6         |

# Identity and Access Management (IAM)

IAM **creates** and **manages** users and groups, using **permissions** to allow and deny access to AWS resources.  
Terms:

- **Policies** \- JSON docs granting permissions for a specific **user**, **group** or **role** to access services. Attached to IAM **identities**.
  - Types:
    - Managed \- Managed by AWS, cannot edit. Have orange box next to name.
    - Customer Managed \- Customer created and editable.
    - Inline \- Policy attached directly to user, not reusable.
  - Made up of:
    - Version \- policy language version, not version of policy
    - Statement \- container for policy element (can have multiple), contains elements below
    - Sid (optional) \- label for statement
    - Effect \- whether policy will Allow or Deny
    - Action \- list of actions to allow / deny
    - Principle \- account, user, role or federated user to apply to
    - Resource \- relevant resource
    - Condition (optional) \- circumstances for permission activation
- **Permission** \- API actions that can or cannot be performed. Represented in IAM Policy document
- **Identities**:
  - **Users** \- End users logging into the console or interacting with AWS resources programmatically or via the UI.
  - **Groups** \- Sets of multiple users so they share permissions levels (e.g. Admins, Devs, Auditors)
  - **Roles** \- Grant AWS resource permissions to specific API actions. Policies associated with a role and then assigned to resource

## Principle of Least Privilege (PoLP)

Computer security concept of providing a user, role or application the **lowest** level of permission possible to achieve an operation or action.  
Contributing concepts:

- Just-Enough-Access (**JEA**) \- only providing permissions for the exact necessary **actions** to perform the required task
- Just-In-Time (**JIT**) \- only providing permissions for the **smallest duration** necessary to perform the required task
- Risk-based adaptive policies \- Any attempt to access a resource generates a **risk score** based on device, IP address, time and much more
  - AWS does **not** have any built in risk-based adaptive policies

## AWS Account Root User

AWS **Account** \= the account holding all the AWS resources  
AWS Account **User** \= user for common tasks assigned permissions  
AWS Account **Root** User \= special account with **full** access

- **Cannot** be deleted
- Uses email and password to log in
- **Full** permissions to account, **cannot** be limited (except by organizations service control policy (SCP)
- **One** root per **account**
- Recommended never use root user access keys
- Recommended MFA for root user

Use root user to:

- **Change account settings**
  - Account name, email address, root user password, root user access keys
  - Contact info, payment currency, regions
- Restore IAM user permissions
- Activate IAM access to Billing and Cost Management console
- View tax invoices
- **Close AWS account**
- **Change or cancel AWS Support plan**
- Register as Reserved Instance Marketplace seller
- Enable MFA delete on S3 buckets
- Edit or delete S3 bucket policy including invalid VPC or VPC endpoint IDs
- Sign up for GovCloud
- Create organization

In IAM you can set **Password Policies** to enforce **minimum requirements** and **rotation** of passwords.

## Temporary Security Credentials

Just like access keys but only last from **minutes** up to an **hour**. Not stored with the user but generated dynamically and provided when requested. Used under the hood for roles and identity federation.  
Used for:

- **Identity federation**
  - Linking electronic identity across identity management systems (e.g. logging in via Google account)
  - Enterprise identity federation (SAML, custom federation brokers)
  - Web identity federations (Amazon, Facebook, Google, OpenID Connect \[OICD\])
- **Delegation**
- **Cross-account** access
  - Allow users from other AWS accounts to assume roles in your account with access to AWS resources
  - STS **AssumeRole**
  - STS **AssumeRoleWithWebIdentity**
    1. Use OAuth to log into third party identity service
    2. Get JWT token back on success
    3. Call STS AssumeRoleWithWebIdentity passing JWT token
    4. Get back temporary credentials on success
    5. Use these to access resources using cross-account role
- **IAM roles**

## AWS Single-Sign on (SSO)

Create or connect workforce identities in AWS once and manage access central across organization. Managed user permissions centrally for AWS accounts, AWS applications and SAML applications.  
Identity sources:

- AWS SSO
- Active Directory
- SAML

# Elastic Compute Cloud (EC2)

Elastic Compute Cloud (**EC2**) is a **configurable** and **resizable** virtual **server**, which can spin up new instances in minutes. Basically everything on AWS uses EC2s under the hood.  
Process:

1. Choose an **OS** via Amazon Machine Image (**AMI**)
2. Choose an instance **type** (varying costs, processing power, memory etc)
3. Choose **storage** (EBS, EFS) \- SSD, HDD, etc.
4. **Configure** instance (security groups, key pairs, IAM roles, etc)

## Cloud-Init

Industry standard multi-distribution method for cross-platform cloud instance initialisation (preparing an instance with config data for OS and runtime env), supported across all major public cloud providers and provisioning systems for private cloud infrastructure.  
Cloud instances are initialised from a disk image and instance data consisting of:

- **Metadata**
  - Access from within EC2 instance, via Metadata Service (MDS/IMDS) endpoint
    - IPv4 \= http\://**169.254.169.254**/latest/meta-data/
    - IPv6 \= http\://**\[fd00:ec2::245\]**/latest/meta-data/
  - Two versions of MDS:
    - IMDSv**1** \- request/response method
    - IMDSv**2** \- session-oriented method (implemented after exploit on IMDSv1 for additional security)
  - Grouped into 60+ categories \- **must** add category on end of request to get metadata
  - Can config instance metadata to:
    - Enforce use of tokens (IMDSv**2**)
      - Will get 401 response if missing
    - Turn off endpoint completely
    - Specify max network hops allowed
- **User data** \- script to run when booting (e.g. install Apache web server)
  - Script must be base64 if directly using the API (CLI and console will automatically encode)
- **Vendor data**

Cloud init supported by most Linux distributions.

## Instance Types

Naming convention:

1. Instance **Family** \- Intended workflow instance type was designed to meet

- **General** Purpose
  - Balance of compute, memory and networking resources
  - A, **T** (cheapest ‘burst’), **M** (even balance), Mac
  - Web servers, code repos etc.
- **Compute** Optimized
  - High performance processor
  - C
  - Modelling, gaming servers etc.
- **Memory** Optimised
  - Fast performance for large data set processing in memory
  - R, X, z, High Memory
  - Memory caches, in-memory DBs, real-time big data analytics
- **Accelerated** Optimised
  - Hardware accelerators or co-processors
  - P, G, F, Inf, VT
  - Machine learning, computational finance etc.
- **Storage** Optimized
  - Fast sequential read and write access to large data sets on local storage
  - I, D, H
  - NoSQL, data warehousing etc.

2. Instance **Generation** \- Version of the instance family
3. **Processor** Family \- Type of processor used

- Intel \- various types
- AMD \- usually similar to intel, trying to undercut
- NVIDIA GPUs \- graphic intense workloads or machine learning
- AWS Graviton \- ARM architecture
- AWS Inferentia \- Cheap, high performance ML inference
- AWS Trainium \- Cheap ML model training
- Various types optimised for different purposes at different prices

4. **Additional Capabilities** \- Extra features the instance type has (e.g. extra storage)
5. Instance **Size** \- The amount of available virtual resources (e.g. CPU, RAM)

- **Roughly** double resources and cost for each jump up

In the format: **1234.5555**… (e.g. c7gn.xlarge)  
Does **not** have to feature every part (e.g. t3.micro)  
Instance families often called instance types but instance type is combo of size and family.

## Instance Profile

IAM role that is passed and assumed by the EC2 instance when it starts up, to avoid passing long life AWS credentials.  
Instance profiles:

- Can be associated at **time of launch** or on a **running** instance
  - Hard reboot is required if none were attached prior
  - Changing roles **not** instant (eventual consistency)
    - Dissociating & reassociating or hard reboot will work
- Are automatically created when selecting an IAM role during EC2 instance launch
  - Not easily viewable from the console

## Instance Lifecycle

**Actions**:

- Launch \- create and start instance based on AMI
- Stop \- turn off but not delete instance
- Start \- turn on a previously stopped instance
- Terminate \- delete instance
- Reboot \- soft reboot at OS level
- Retire \- notifies an instance scheduled for retirement
- Recover \- automatically recovered failed instance on new hardware if enabled (keeps instance ID and other config)

**States**:

- Pending \- preparing to enter running state
- Running \- instance active and ready for use
- Stopping \- in process of becoming stopped
- Stopped \- Inactive and not usable, ready to be restarted
- Shutting down \- preparing for termination
- Terminated \- permanently deleted, cannot be restarted

## Instance Console Screenshot

Instance Console Screenshot will take a screenshot of the current state of the instance in order to troubleshoot booting issues. Also possible via the CLI. That’s all.

## Hostnames

Identify a machine in a network via a unique name using DNS. Useful for when software is expecting a very specific name for example some micro-service architectures. Must ensure cloud init config will preserve hostnames and reboot after the change.  
**Formats**:

- **IP** Name \- based on private IPv4 address
  - IPv**4** only instances (or choosing in dualstack)
  - **us-east-1** \= \[private-ipv4-address\].ec2.internal (e.g. ip-10-24-34-0.ec2.internal)
  - **Other regions** \= \[private-ipv4-address\].region.compute.internal (e.g. ip-10-24-34-0.ca-central-1.compute.internal)
- **Resource** Name \- based on instance ID
  - IPv**6** only instances (or choosing in dualstack)
  - **us-east-1** \= \[ec2-instance-id\].ec2.internal (e.g. i-0123456789abcdef.ec2.internal)
  - **Other regions** \= \[private-ipv4-address\].region.compute.internal (e.g. i-0123456789abcdef.ca-central-1.compute.internal)

## Default User Name

Default user name for OS is useful to know to use sessions manager or SSH on EC2 instance. Almost always **ec2-user** but can vary (e.g. ubuntu, bitnami, admin, root).

## Burstable Instances

Can use more hardware resources to cope with sudden spikes in usage for short durations without having to upgrade overall. **T** family. Can be able to spend just accumulated CPU credits (**standard** mode) or go beyond this for additional charge (**unlimited** mode). At least one of these models should be free tier.

## Source & Destination Checks

The source/destination check restricts an instance to only send or receive traffic when it is the source or destination. Helpful to not be used as a middle man for nefarious activities. Should be disabled in many cases such as using an instance for NAT.

## Placement Groups

Allow logical organisation of instances to optimise communication, performance and durability.

- **Cluster**
  - Packs instances closely together
  - Ideal for tightly-coupled node communication or high performance computing
  - Can**not** be **multi-AZ**
- **Partition**
  - Spread instances over **multiple** logical **partitions** which do not share underlying hardware
  - Ideal for large distributed / replicated workloads (e.g. Kafka, Cassandra)
- **Spread**
  - Each instance has its own underlying hardware (rack)
  - Ideal for separate critical instances
  - Max of **7** instances
  - Can be **multi-AZ**

## Connecting to EC2

Connection methods:

- **SSH**
  - Connect from local machine using public and private key
  - Public and private key generated on AWS
  - **Port** **22** must be open on the **Security Group**
- EC2 **Instance Connect**
  - Short-lived SSH keys controlled by IAM Policies
  - Only works on **Linux** and not on all instances
- **Sessions Manager**
  - Reverse connection
  - Works for **Windows** and **Linux**
    - Windows \-\> Powershell
    - Linux \-\> Bash shell
  - Access controlled by **IAM**
  - Supports login audit trails
- **Fleet Manager Remoted Desktop**
  - Works on **Windows**
  - Connect via **RDP** within web browser
- EC2 **Serial console**
  - Serial connection for troubleshooting **hardware**

## Amazon Linux

AWS’ managed Linux distribution based off CentOS and Fedora (based off Red Hat Linux (RHEL). It has the best support and often is the only choice for underlying service OS. Better technical support is given for it than other OS. Yum is package manager used for earlier versions, improved ‘dnf’ manager available in AL2023.  
Versions:

- Amazon Linux 1 (**AL1**) \- deprecated
- Amazon Linux 2 (**AL2**) \- deprecated \<check if deprecation went ahead and if AL2 specific content relevant now\>
- **Amazon Linux 2023** (AL2023)
  - Contains many packages built in
  - Python 3 default
  - Security Enhanced Linux (SELinux) enabled by default
  - OpenSSL 3
  - Source CentOS9 stream
  - Gp3 volumes by default
  - Cronie not installed by default
  - Amazon Corretto for Java RTE

## AMI

An Amazon Machine Image (**AMI**) is an instance template. You can create an AMI from an EC2 instance to create copies of your server. You can copy an AMI (**even to another region**). You can encrypt a non-encrypted AMI during copy. AMIs can also be stored in, and restored from, S3 buckets. This can copy AMIs **across partitions**.  
Contains:

- **Root volume template** \- EBS snapshot or Instance Store Template (e.g. OS, application server, applications)
- **Launch permissions** \- Control which AWS accounts can use AMI to launch instances
- **Block device mapping** \- Specify which volumes to attach to the instance once launched

AMIs are **region-specific**. AMIs are useful to keep incremental changes to your OS, applications and packages. **Systems Manager Automation** can routinely patch AMIs with security updates and ‘**bake**’ those AMIs. Launch Configurations or **Launch Templates** (newer) manage AMI revisions. Can get AMIs on the **Marketplace**.  
Can be selected based on:

- Region
- OS
- Architecture (32/64 bit, ARM, etc)
- Launch permissions
- Root device volume
  - **Instance store** (ephemeral storage)
    - Native instance volumes used
    - Data lost on instance stop
  - **EBS-backed volumes**
    - EBS storage automatically attached at launch
    - Independent of instance (can be stopped and restarted without data loss)

Two boot modes:

1. **Legacy BIOS** (Basic Input-Output System) \- traditional interface used for decades

- No support for secure boot
- May be required in some cases (e.g. legacy OS or application)

2. Unified Extensible Firmware Interface (**UEFI**) \- modern interface to replace legacy BIOS

- Supports secure boot
- Faster startup times
- Supports drives \>2TB
- Pre-boot environment with some graphical UI and networking capabilities

Use 2 unless there’s a good reason.  
Some AMIs are **Elastic Network Adaptor** (ENA) enabled, which can support up to 100Gb/s. All nitro-based instances use ENA for advanced networking.  
AMIs can be:

- **Deregistered** \- no more instances launched from AMI (does not delete snapshot)
- **Deprecated** \- mark date when use no longer allowed
- **Disabled** \- prevent from being used. Can be re-enabled later
- **Shared** \- expand the ability to launch instances from AMI to AWS accounts
  - Public \- any AWS account can launch
  - Explicit \- specific AWS accounts, organisations or organisational units can launch
  - Implicit (default) \- the owner can launch

Virtualisation Types:

|                       | Hardware Virtual Machine (HVM)                                                    | Paravirtualisation (PV)                                                                   |
| :-------------------- | :-------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------- |
| **Virtualisation**    | Full virtualisation with hardware assistance                                      | Software-assisted virtualisation requiring OS modification                                |
| **Hardware Assisted** | Uses hardware-assist technology from the host system’s CPU                        | Relies on a hypervisor to simulate hardware                                               |
| **Performance**       | Potentially higher, especially for applications needing direct access to hardware | Initially better for many applications due to reduced overhead but has now been surpassed |
| **OS Support**        | Broader \- can run any OS that can be run on physical hardware                    | Limited to only modified OS                                                               |
| **Boot Method**       | Can boot from EBS volume or instance store                                        | Can only boot from instance store                                                         |
| **Usage**             | Modern OS and applications requiring specific hardware features                   | Historically used for certain workloads and older instance types. Not used much now.      |

PV basically retired.

## EC2 Pricing Models

- On-Demand \- least commitment
  - Default from launch now
  - Low cost
  - Flexible
  - Only pay by the hour or second
  - Short-term unpredictable work loads
  - Cannot be interrupted
  - First time apps
- Spot \- biggest savings (up to 90%)
  - Request spare computing capacity
  - Flexible start and end times
  - Can handle interruptions \- server randomly stopping and starting)
  - For non-critical background jobs
  - Good for AWS Batch
- Reserved \- best long term (up to 75% off)
  - Stead, predictable usages
  - Commit over 1 or 3 year term
    - Not auto-renew
    - Will go to on-demand after term
  - Can resell unused reserved instances (RIs) in the Reserved Instance Marketplace
  - Pricing \= Term x Class x Attributes x Payment option
    - Term \= length of commitment
    - Class \= type of RI
      - Standard \- up to 75% off
        - Can modify RI attributes:
          - Change AZ within same region
          - Change scope of zonal RI to regional or vice versa
          - Change instance size
          - Change network EC2-classic \-\> VPC and vice versa
        - Cannot be exchanged
        - Can be bought or sold in RI marketplace
      - Convertible \- up to 54% off
        - RI attributes can’t be modified (perform exchange instead)
        - Can be exchanged for another convertible RI with new attributes
        - Can’t be bought or sold on RI marketplace
      - ~~Scheduling~~
    - Instance attributes:
      - Instance type \- t3.micro, m4.large etc
      - Region
      - Tenancy \- shared or single-tenant
      - Platform \- OS
    - Payment options:
      - All upfront \- full payment made at start
      - Partial upfront \- portion paid upfront, rest hourly at discounted rate
      - No upfront \- discounted hourly rate regardless of whether instance is in use
  - RIs can be shared between multiple accounts in the same AWS Organisation
  - Scope must be defined when purchasing RI
    - Does not affect price
    - Select either:
      - A region
        - Does not reserve capacity
        - Discount applies to instance usage in an AZ in the region
        - Instance size flexibility
        - Can queue purchases
      - An AZ
        - Reserves capacity in that AZ
        - No AZ flexibility
        - No size flexibility
        - Can’t queue purchases
  - RI limits:
    - 20 regional RIs per region per month
    - 20 zonal RIs per AZ per month
    - Can exceed on-demand limit by purchasing zonal RIs
      - Cannot do this with regional RIs
  - Capacity reservations
    - Request a reserved instance type for region and AQ
    - Charged for it whether instance is running or not
    - Always available
  - RI Marketplace
    - Instances can be sold after being active 30 days and any upfront payment being made
    - Must have US bank account to sell
    - Retain pricing and capacity until sold
    - Seller can only set upfront price
      - Usage and other config will stay the same
    - Term length rounded down to nearest month
    - Sell up to \$20k per year
    - Instances in GovCloud region cannot be sold
- Dedicated \- most expensive
  - Dedicated servers
  - Can be on-demand, reserved or spot
  - Guarantee of isolate hardware
  - Enterprise

AWS Savings Plan is another way to save, but wider than just EC2.  
Types:

- Compute savings plan
  - Most flexibility
  - Reduce costs by up to 66%
  - Automatically apply to EC2 instance, Fargate and Lambda usage
    - Regardless of instance family, size, AZ, region, OS or tenancy
- EC2 instance savings plan
  - Lowest prices
  - Reduce costs up to 72%
  - Automatically reduces cost on selected instance family
    - Regardless of size, AZ, region, OS or tenancy
  - Flexibility to change usage between instances within a family in that region
- SageMaker savings plan
  - Helps reduce SageMaker costs by up to 64%
  - Automatically applied to SageMaker usage regardless of instance family, size, component or region

Terms:

- 1 year
- 3 years

Payment options:

- All upfront
- Partial upfront
- No upfront

Choose hourly commitment spend.

# Auto-Scaling Groups (ASG)

A collection of EC2 instances which are automatically scaled **horizontally** together according to their group configuration:

- **Capacity Settings** (**Manual** Scaling) \- set the expected range of capacity
  - **Min** size \- the **lowest** acceptable number of running EC2 instances
  - **Max** size \- the **highest** allowed number of running EC2 instances
  - Desired capacity \- the **ideal** number of running EC2 instances
  - ASG will meet **min** size of instances on launch
- **Health Check Replacements** \- replace instances if they are deemed unhealthy
  - EC2 health check \- EC2 instance status check
  - ELB health check \- ELB pings HTTP endpoint at specific path, port and status code
- **Scaling Policies** (**Dynamic** Scaling) \- complex rules to determine when to scale up or down
  - Simple scaling
    - Change capacity in either direction by a specified amount when a **CloudWatch Alarm** is triggered
    - Cooldown period is recommended
    - Using step or target tracking scaling instead is recommended
  - Step scaling
    - Change capacity in either direction by specified amounts at different thresholds (steps) when a **CloudWatch Alarm** is **repeatedly** triggered
  - Target Tracking scaling
    - Automatically changes capacity in either direction to attempt to **meet** a **target metric** provided
    - E.g.
      - Average CPU Utilisation
      - Average requests per second
      - Average network traffic in
      - Average network traffic out
      - Custom metrics
    - Will create 2 CloudWatch alarms for you
  - Predictive scaling
    - Automatically changes capacity in either direction based on analysis of **historical load** on **target metric axes** to detect daily or weekly patterns
    - Need a 24h forecast of CloudWatch data before you can activate
    - Will continuously use the last **14 days** of data to make adjustments
    - Will produce **hourly** forecasts for capacity requirements over the next **48h**
    - Will update every **6h** using latest CloudWatch data

**ASG**s used to scale EC2s \- ECS or EKS using EC2 instances will both work. Fargate does **not** use ASGs.  
**Termination Policies** determine the order for terminating instances. AWS provides predefined policies (e.g. oldest instance first), but custom termination policies can be created by invoking **Lambda** functions.

## ELB-ASG Integration

An **Elastic Load Balance**r can be attached to an **Auto-Scaling Group** to allow the ASG to use the ELB health check on instances. Classic load balancers are associated **directly** with ASGs \- ALBs, NLBs and GWLBs are associated **indirectly** via their **Target Groups**.

## Elastic Load Balancer (ELB)

## Load Balancer Types

Load Balancers are hardware or software that accepts incoming traffic and routes it on to multiple targets using various balancing rules. Elastic Load Balancer (ELB) is a suite of load balancers used to distribute traffic to multiple EC2, ECS, EKS or Fargate instances.  
Load balancer types:

- **Application Load Balancer (ALB)**
  - Application layer (OSI level 7\) (HTTP/HTTPS)
  - Routes based on HTTP info
  - Can leverage Web Application Firewall (WAF)
  - Has Request Routing (allows adding routing rules based on HTTP)
  - Supports WebSockets and HTTP/2 for realtime, bidirectional communication applications
  - Can handle authentication and authorisation of requests
  - Can only be accessed via hostname
    - If static IP needed, forward NLB to ALB
  - AWS Certificate Manager (ACM) can be attached to listeners to server custom domains over SSL/TLS for HTTPS
  - Global Accelerator can be placed in front to improve global availability
  - Amazon CloudFront can be placed in front to improve global caching of common HTTP requests
  - Amazon Cognito can be used to authenticate users via incoming HTTP requests
  - Use cases:
    - Microservices and containerised applications
    - E-commerce and retail websites
    - Corporate websites and applications
    - SaaS applications
- **Network Load Balancer (NLB)**
  - OSI layer 3/4 (TCP/UDP)
  - Designed for large throughput of low-level traffic
  - Can handle millions of requests per second with very low latency
  - Global Accelerator can be placed in front for improved global availability
  - Preserves client source IP
  - Use cases:
    - When static IP needed for LB
    - High-performance computing
    - Big data applications
    - Realtime gaming platforms
    - Financial trading platforms
    - Telecoms network
- **Gateway Load Balancer (GWLB)**
  - Routes traffic through virtual appliances before reaching destination
  - Useful as a security level
- **Classic Load Balancer (CLB)**
  - OSI level 7 and 3/4
  - Doesn’t use target groups \- directly attaches targets
  - Previous generation of LB only used in legacy cases

Listeners evaluate incoming traffic on their port. Listeners will then invoke rules (OSI 7 only) to decide what to do with traffic. Usually this will be to forward on to target groups. Target groups are a logical grouping of possible targets (e.g. EC2 instances, IP addresses). CLB just directly associates targets with the load balancers instead.

## Route53

Domain Name Service (**DNS**) with integrations with AWS services.  
Can:

- Register & manage domains
- Create record sets on domains
- Implement complex traffic flows
- Monitor records via Health Checks
- Resolve VPCs outside of AWS

**Traffic flow** is a UI workflow supporting creation of sophisticated routing configurations as well as versioning (\$50/month per policy record).

### Hosted Zone

Hosted Zone in Route 53 is a container for record sets, scoped to route traffic for specific domains (e.g. recordname.app.saas) or subdomains (e.g. app.recordname).  
Two types:

- **Public** \- how to route inbound traffic from the internet
- **Private** \- how to route traffic from within an Amazon VPC

### Records

Record Sets are collections of records determining where to send traffic. Changed in batch via the API, with CREATE, DELETE and UPSERT operations. Alias records route traffic to specific AWS endpoints dynamically (reference will survive changes in resource IP). Using alias records is recommended when routing to AWS resources like:

- **CloudFront** (d1234567abcdef8.cloudfront.net)
- **Elastic Beanstalk** env (example.elasticbeanstalk.com)
- **Elastic Load Balancer** (example-1.us-east-2.elb.amazonaws.com)
- **S3** website endpoint (s3-website.us-east-2.amazonaws.com)
- Resource record set (www\.example.com)
- **VPC** endpoint (example.us-east-2.vpc2.amazonaws.com)
- **API** **gateway** endpoint custom regional API (d-abcde-1234.execute-api.us-west-2.amazonaws.com)

### Routing Policies

7 types:

- **Simple** routing (default)
  - Multiple IP addresses per record
  - Traffic routed to one of IP addresses at random
- **Weighted** routing
  - Multiple IP addresses per record
  - Relative weight per address
  - Random routing with frequency correlating with relative weights
- **Latency-based** routing
  - Routing based on lowest-latency region
  - Requires latency resource record for each application-hosting resource in each region
  - Example: 2 copies of 1 application backed by ALB, hosted in different regions and always routed to lower latency of the two as seen by the user
  - Usually proximity-based but not in principle
- **Failover** routing
  - Route 53 will monitor health checks on primary route
  - Will begin routing to secondary location when health checks on primary fail
- **Geolocation** routing
  - Directs traffic based on location (e.g. all north america traffic to us-east-1)
- **Geo-proximity** routing
  - Only available via Traffic Flow
  - Gives different regions different biases (effectively relative ranges)
  - Traffic routed based on location according to which bias-adjusted region they are located in
- **Multi-value answer** routing
  - Same as simple routing but with added health check on targets

### Health Checks

Checks the correct functioning and health of AWS endpoints. Can have **up to 50** health checks for endpoints within, or linked to, an AWS account. Checks performed **every 30s** by default, can be reduced to **10s**. If unhealthy, can initiate a failover or create CloudWatch alarm to alert. Can also chain health checks together by monitoring others in a health check.

### Resolver

Route 53 Resolver (formerly .2 Resolver and Amazon DNS Server) is a DNS server allowing the resolution of DNS queries between an on-premise DNS resolver and a VPC. **Inbound** resolver endpoints allow DNS queries to the VPC from an on-premise network or other VPC. **Outbound** resolver endpoints allow DNS queries from the VPC to an on-premise network or other VPC.

### DNSSEC

Domain Name System Security Extensions (DNSSEC) are a suite of extension specifications by the Internet Engineering Task Force (IETF) for securing data exchanged in the Domain Name System (DNS) in Internet Protocol (IP) networks. DNSSEC signing allows DNS resolvers to validate that a DNS response came from Route 53 and has not been tampered with. It’s important to enable DNSSEC on your domains so others cannot impersonate them. Involves signing with KSK key.

### Zonal Shift

Capability in Amazon Route 53 Application Recovery Controller (ARC) that shifts a load balancer resource away from an impaired availability zone towards a healthy one.  
Conditions:

- Only supported on Application Load Balancers (**ALB**s) and Network Load Balancers (**NLB**s) with cross-zone load balancing turned off
- **Not** supported when using ALB as an accelerator endpoint in AWS Global Accelerator
- Only for **single AZ** per load balancer

### Route 53 Profiles

Profiles let you manage and apply Route 53 DNS-related configurations across different VPCs and AWS accounts.  
You can associate:

- Private hosted zones
- Resolver rules
- DNS firewall rule groups

## AWS Global Accelerator

Can find the optimal path from an end user to one of your web servers. Deployed within **Edge Locations** so traffic is sent here, not directly to the web application. Has a speed comparison tool.  
Types:

- **Standard** \- automatically route to nearest healthy endpoint
- **Custom** Routing \- route to specific EC2 instances

Components:

- **Listeners** (e.g. TCP 3000\) \- listen for traffic on a specific port and sends to an endpoint group
- **Endpoint Groups** \- collection of Endpoints within a specific region. A Traffic Dial can be used to balance traffic loads
- **Endpoints** \- resources to send traffic to. Can be:
  - Network Load Balancer (NLB)
  - Application Load Balancer (ALB)
  - EC2 instance
  - Elastic IP address

## CloudFront

A Content Delivery Network (**CDN**) is a distributed network of servers that provide web pages or other content to users based on geographical location, the origin of the web page and a content delivery server. Essentially caching across a distributed network strategically for service speed.  
CloudFront is a CDN that can be used to deliver:

- Static content
- Dynamic content
- Video streaming
- Web sockets

Components:

- **Origin** \- location where all original files are stored (e.g. S3 bucket, EC2 instance, ELB or Route 53\)
  - Domain name \- e.g. ‘mybucket.s3.amazonaws.com’
  - Origin path (optional) \- e.g. ‘/accounts’
  - S3OriginConfig
  - CustomOriginConfig (non-s3)
- **Edge Location** \- compute located strategically close to end user where copies are cached
- **Regional Cache** \- computed located in broad geographical regions to speed up requests for edge locations
- **Distribution** \- collection of edge locations and regional caches that defines how cached content should behave

CloudFront can:

- Be fronted with AWS WAF for OWASP top 10 protection
- Stream videos on demand using ISS Microsoft Smooth Streaming

## Lambda@Edge

Lambda functions to override the behavior of requests and responses operating via CloudFront.  
Function types (lifecycle order):

1. **Viewer Request** \- when CloudFront receives a request from a viewer

   Use cases:

- Redirect HTTP \-\> HTTPS (CloudFront could just do this)
- Inspect cookies for user authentication
- Modify headers for A/B testing

2. **Origin Request** \- before CloudFront forwards a request to the origin

Use cases:

- Rewrite URLs for SEO or routing
- Inject headers for origin authentication
- Selective content serving based on user-agent (e.g. mobile vs. desktop)

3. **Origin Response** \- when CloudFront receives a response from the origin

Use cases:

- Modify headers to control caching
- Update URLs in HTML for versioning
- Customise error responses from origin

4. **Viewer Response** \- before CloudFront forwards a response to the viewer

   Use cases:

- Add security headers (e.g. CSP, HSTS)
- Set cookies for client-side tracking
- Customise error messages

Functions are deployed at **regional** edge cache level. Supported languages are Python and Node.js.

## CloudFront Functions

Lightweight edge functions for high-scale, latency-sensitive CDN customisations. **Cheaper** and **faster**, but **more limited** than, Lambda@Edge functions.  
Functions (lifecycle order):

1. **Viewer Request** \- when CloudFront receives a request from a viewer
2. **Viewer Response** \- Before CloudFront returns the response to the viewer

Functions are deployed at **edge locations**. Only Javascript currently supported \<check\>.  
Use cases:

- Cache key normalisation
- Header manipulation
- Status code modification and body generation
- URL redirects or rewrites
- Request authorisation

## Lambda@Edge vs. CloudFront Functions

| Category                                     | Lambda@Edge functions                                                    | CloudFront Functions                    |
| -------------------------------------------- | ------------------------------------------------------------------------ | --------------------------------------- |
| Scale                                        | Up to 10,000 request per second per region                               | 10,000,000 requests per second or more  |
| Function duration                            | Viewer request/response \= up to 5s Origin request/response \= up to 30s | \< 1 millisecond                        |
| Max memory                                   | 128-3008 MB                                                              | 2 MB                                    |
| Max code \+ libs size                        | Viewer request/response \= 1MB Origin request/response \= 50MB           | 10 KB                                   |
| Network, file system and request body access | Yes                                                                      | No                                      |
| Geolocation and device data access           | Viewer request \= No Viewer response, origin request/response \= Yes     | Yes                                     |
| Can build & test in CloudFront               | No                                                                       | Yes                                     |
| Function logging & metrics                   | Yes                                                                      | Yes                                     |
| Pricing                                      | Costs per request & function duration                                    | Costs per request (free tier available) |

## Elastic Block Store (EBS)

**IOPS** \= input/output per second. Speed of non-contiguous reads and writes that can be performed on storage medium.  
**Throughput** \= data transfer rate to and from storage medium in Mb/s.  
**Bandwidth** \= total possible speed of data movement along network.  
Bandwidth \= pipe, throughput \= water.

**EBS** is a highly available and durable solution for attaching persistent block storage volumes to an EC2 instance.  
Volumes automatically replicated within their AZ to protect from failure.  
Types:

- **General** **Purpose** SSD
  - Gp2 \- general usage without specific requirements
  - Gp3 \- up to 20% lower cost per GB than gp2
- **Provisioned** IOPS SSD
  - Io1 \- for fast input/output requirements
  - Io2 \- more durable than io1 (retired for block express)
  - Io2 Block Express \- higher throughput, IOPS and larger storage capacity
- **Cold** HHD (sc1) \- lowest cost HHD volume for infrequent access
- **Throughput Optimised** HDD (st1) \- magnetic drive optimised for quick throughput
- **Magnetic** \- previous generation HDD

|                      | General Purpose SSD      | Provisioned IOPS SSD                                                                                                                      | Throughput Optimised HHD | Cold HDD                                                                        | Magnetic                        |
| :------------------- | :----------------------- | :---------------------------------------------------------------------------------------------------------------------------------------- | :----------------------- | :------------------------------------------------------------------------------ | :------------------------------ |
| **Use cases**        | General workloads        | Io1 \= Sustained IOPS performance required / \>16k IOPS, I/O-intensive db workloads Io2 \= \<1ms latency, \>64k IOPS, 1000MB/s throughput | Big data, log processing | Throughput-oriented storage infrequently accessed, low storage cost requirement | Data very infrequently accessed |
| **Durability**       | 99.8%-99.9%              | Io1 \= 99.8%-99.8% Io2 \= 99.999%                                                                                                         | 99.8%-99.9%              | 99.8%-99.9%                                                                     | N/A                             |
| **Volume size**      | 1GB \- 16TB              | Io1 \= 4GB-16TB Io2 \= 4GB-64TB                                                                                                           | 125GB-16TB               | 125GB-16TB                                                                      | 1GB-1TB                         |
| **Max IOPS**         | 16KB I/O                 | Io1 \= 64k (16KB I/O) Io2 \= 256k (16KB I/O)                                                                                              | 500 (1MB I/O)            | 250 (1MB I/O)                                                                   | 40-200                          |
| **Max throughput**   | gp2=250MB/s gp3=1000MB/s | Io1 \= 1000 MB/s Io2 \= 4,000 MB/s                                                                                                        | 500 MB/s                 | 250 MB/s                                                                        | 40-90 MB/s                      |
| **EBS multi-attach** | No                       | Yes                                                                                                                                       | No                       | No                                                                              | N/A                             |
| **NVMe reserve**     | No                       | Io1 \= No Io2 \= Yes                                                                                                                      | N/A                      | N/A                                                                             | N/A                             |
| **Boot volume**      | Yes                      | Yes                                                                                                                                       | No                       | No                                                                              | Yes                             |

### Hard Disk Drives (HDD)

Magnetic storage using rotation to access disk area. Good at writing continuous data, not good for many small read/write operations.  
RPMs:

- 5400 \- lower power consumption & heat priorities over performance (e.g. laptops)
- 7200 \- standard balance of performance and cost (e.g. desktops)
- 10000 \- performance-critical, usually now SSD (e.g. enterprise workstations)

### Redundant Array of Independent Disks (RAID)

Data storage virtual technology for magnetic disks to improve fault tolerance, combining **multiple** physical **volumes** into **one** logical **group** to overcome wear in HDD.  
Types:

- RAID **0** (Striping)
  - No redundancy \- data split across disks for performance
  - Minimum 2 disks
- RAID **1** (Mirroring)
  - Duplication across disks
  - Minimum 2 disks
- RAID **5** (Striping \+ parity)
  - Speed and data protection
  - Minimum 3 disks
- RAID **6** (Striping \+ double parity)
  - RAID 5 with extra parity for double failure tolerance
  - Minimum 4 disks
- RAID **10** (1+0)
  - Combination of RAID 0 and RAID 1 for redundancy and performance
  - Minimum 4 disks

### Solid State Drives (SSD)

Integrated Circuit (**IC**) assemblies as memory storage, usually using flash memory.  
Resistant to shock, run silently and have quick access time and low latency. Good for **frequent I/O**, no moving parts.  
Types:

- SATA \- widely used and compatible, good performance generally
- **NVMe** \- higher performance for intensive data tasks
  - Use PCIe interface
- M.2 \- compact and installed directly in motherboard, good for laptops etc.
  - Can use SATA or NVMe interfaces
- U.2 \- similar performance to M.2, mostly used in enterprise and server environments
- Portable \- external drives for portability, connected via USB or Thunderbolt
- PCIe \- add-on cards providing high performance for older systems or specialised tasks

### Elastic File System (EFS)

File storage service for EC2 instances where **capacity grows and shrinks** automatically based on data volume. Multiple EC2 instances in the same VPC can mount to a single EFS volume (must be in **same** **VPC**). Instances install NFS client and then mount EFS volume. EFS creates multiple mount targets in all VPC subnets. Charged by space used starting at \$0.30 per GB per month. Can also be mounted to Lambda and FARGATE.

#### Amazon EFS Client

Open-source collection of EFS tools enabling ability to use CloudWatch to monitor EFS file system’s mount status. Need to install on EC2 instance prior to mounting EFS file system. Includes mount helper. Can mount Linux or Mac.

### Amazon FSx

Allows deployment and scaling of feature-rich, high-performance file systems in the cloud.  
Types:

- NetApp ONTAP \- proprietary enterprise storage platform handling petabytes of data
- OpenZFS \- open-source storage platform originally developed by Sun Microsystems
- Windows File Server (WFS) \- storage on a Windows server for Windows developers
  - Sub-millisecond latencies
  - Offers SSD, HDD or both
- Lustre \- Open-source file system for parallel computing

### Amazon File Cache

High-speed cache for datasets stored in on-premises file systems, AWS file systems, S3 buckets, found under the FSx Management Console. Accessible to EC2, ECS and EKS. Compatible with most popular Linux-based AMIs. Makes dispersed datasets available to file-based applications on AWS with unified view at high throughput, high speeds and low latencies.

### AWS Backup

Centrally manage backups across AWS services such as:

- S3
- VMWare VMs
- DynamoDB
- FSx file systems
- EC2
- EFS
- EBS
- RDS & Aurora
- AWS BackInt
- SGW
- DocumentDB
- Neptune
- Timestream

Backup Plan \- backup policy defining backup schedule, window and lifecycle  
Backup Vault \- where backups are stored

- AWS Backup Vault Lock allows for Write-Once-Read-Many (WORM) to set a retention period
- Standard Value (Default) \- initial storage place
- Air-Gapped Vault \- backups can be moved to logically air-gapped vault for additional security

It is possible to:

- Assign resources backup plans using AWS Resource Tags
- Backup resource to other regions or accounts
- Managed backups from a centralised account across entire organisation
- Use an independent KMS encryption key (using AWS Backup)

Backups are incremental, so only differences are stored instead of full copies.  
Charges appear as ‘Backup’ under Cost Explorer.  
AWS Backups are immutable to avoid tampering. AWS Backups have built-in reporting and auditing via Backup Audit Manager.  
Can schedule backups.

### AWS Snow Family

Storage and compute devices used to physically move data in or out of the cloud when moving data over the internet or a private connection is too slow, costly or otherwise difficult. Data delivered to S3 (or glacier for snowmobile). Physically-shipped data \- small parcel, briefcase or literal truck.  
Types:

- **Snowcone**
  - 8TB HHD
  - 14TB SSD
- **Snowball Edge**
  - Storage optimised (80/210TB)
  - Compute optimised (39.5TB)
- **Snowmobile**
  - 100PB

### AWS Transfer Families

Fully-managed support for the transfer of files into or out of S3 or EFS over various protocols:

- File Transfer Protocol (**FTP**) \- what it sounds like
- Secure File Transfer Protocol (**SFTP**) \- FTP with encryption using SSH
- FTP Secure (**FTPS**) \- extends FTP with SSL/TLS encryption
- Applicability Statement 2 (**AS2**) \- enables secure and reliable messaging over HTTP/S. Used in e-commerce etc that require proof of compliant data transfers

Common ports:

- FTP \- **20** (control commands), **21** (data transfer)
- SFTP \- **22**
- FTPS \- **990**
- AS2 \- **443**

Transfer Family Managed File Transfer Workflows (**MFTW**) is a fully managed, serverless file transfer workflow service to set up, run, automate and monitor the processing of files uploaded via AWS Transfer Family.  
Operations:

- Copy \- copy to another S3 destination
- Tag \- apply metadata tagging
- Delete \- delete
- Custom file-processing step \- pass file to a lambda
- Decrypt \- automatically descrypt file using PGP after uploading

### AWS Migration Hub

Single place to discover existing servers, plan migrations and track their status when in process.  
Can monitor migration statuses from Application Migration Service (AMS) and Database Migration Service (DMS).  
Discovery Agent \- agent installed on VM of servers to help discovery  
Migration Evaluator Collector \- submit request to AWS to help assess a migration  
Migration Hub Refactor \- bridges networking across AWS accounts so legacy and new services can communicate while maintaining account independence.  
Migration Hub Journey \- guided templates for end-to-end migrations.

### AWS DataSync

Data transfer service simplifying data migration between cloud storage services over the following protocols:

- Network File System (NFS)
- Server Message Block (SMB)
- Hadoop Distributed File Systems (HDFS)
- Object storage

Works with AWS services:

- S3
- EFS
- FSx
- Snowcone
- S3 compatible snowball edge

Works with other cloud storage services:

- Google Cloud Storage
- Microsoft Azure Blob Storage
- Microsoft Azure Files
- Many more

Can run on a schedule.

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

### AWS Auto Scaling

Service centralising auto scaling resources. Can manage and make recommendations for:

- EC2 Auto Scaling Groups
- ECS EC2 (uses auto scaling groups under the hood)
- Amazon Aurora
- Amazon DynamoDB
- Spot Fleet

Easily apply Dynamic or Predictive scaling.

### AWS Amplify

Opinionated framework and fully-managed infrastructure to allow developers to focus on building web and mobile applications.  
Encompasses:

- Amplify CLI \- unified toolchain to create, manage and integrate AWS cloud services
- Amplify SDK \- connects AWS services to client-side code
- Amplify UI \- accessible, themeable, performant React components directly connected to cloud
- Amplify Hosting \- static website hosting platform
- Amplify Studio \- visual dev environment for building full stack web and mobile apps

Has direct integrations with:

- Amazon Cognito \- for auth signin and signup (must use amplify for this now)
- API Gateway \- for rest APIs
- AppSync \- for GraphQL APIs
- S3 \- for storage or static website hosting
- DynamoDB \- for backend db
- Lambda \- for custom resolvers

Works with js frameworks:

- JavaScript
- React
- Flutter
- Swift
- Android
- React Native
- Angular
- Next.js
- Vue

There is a gen 2 now.

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

### GraphQL

Open-source agnostic query adaptor allowing querying of data from many different data sources. Used to build APIs where clients will send a query for nested data. Mitigated the issue of versioned or rapidly-changing APIs compared to REST because just the wanted data can be requested.  
GraphQL schemas (written in GraphQL Schema Definition Language) composed of:

- Types \- represent objects and their fields
- Fields
- Queries \- define the exact shape of the data needed by the client
- Mutations \- allow for data to be created, updated or deleted
- Subscriptions \- support live updates sent from the server to the client

### AWS AppSync

Fully-managed GraphQL service. Uses Resolvers attached to specific fields within types in the schema to implement state-changing operations for the query, mutation and subscription field operations. Can support real-time APIs via GraphQL subscriptions feature.  
API types:

- GraphQL APIs \- single API from multiple data sources
- Merged APIs \- collection of Graph APIs that act as one

Data sources:

- DynamoDB tables
- Amazon OpenSearch
- AWS Lambda function
- HTTP endpoint
- Amazon EventBridge
- RDS

Caching options:

- None \- resolvers will always fetch from data sources
- Full request caching \- cache all requests
- Per-resolver caching \- specific operation or field defined in a resolver will return responses from the cache

Caching requires the provision of an on-demand instance \- not serverless.  
Authorisation types:

- API key
- AWS IAM
- Amazon Cognito User Pools

AppSync supports custom domains and has a query editor built into the UI.

### AWS Batch

Plans, schedules and executes batch computing workloads across the full range of AWS compute services. Can utilise Spot Instance to save money.  
Definitions:

- Job \- named unit of work (e.g. Shell script, docker container image)
- Job Definition \- defines how to run the job (e.g. amount of compute / memory)
- Job Queue \- collection of jobs that determine job priority
- Job Scheduler \- evaluates when, where and how to run jobs that are submitted to a queue (defaults to FIFO)
- Job Dependencies \- allow specifying a job ID to another job to wait for it before scheduling processing

Batch can run jobs on:

- EC2
- Fargate
- EKS

Job types:

- Array job \- shares common parameters (e.g. job definition, vCPUs, memory)
- Multi-node parallel job \- run single jobs that span multiple EC2 instances
- GPU job \- run on EC2 GPU-based instance types

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

### AWS Device Farm

Application testing service in different real environments \- not simulators.  
Supports:

- Mobile device testing
  - Native iOS / Android / mobile web app
  - Built-in tests Fuzz can randomly test actions
  - Videos captured of runs
  - Various real devices available
  - Can test using Appium suite
- Desktop browser testing
  - Various web browsers available
  - Selenium to write tests

### Amazon Quantum Ledger Database

Fully-managed ledger database providing transparent, immutable and cryptographically viable transaction logs for when you need to record the history of financial activities that can be trusted.  
Features:

- Immutable logs \- data cannot be altered or deleted
- SHA-256 hashing \- secure, verifiable transaction histories
- Fully-managed \- no manual hardware management
- Serverless \- scales automatically
- SQL-like queries \- PartiQL
- High throughput & scalability \- designed for rapid and frequent updates with minimal latency
- AWS integration
- ACID transaction \- atomic, consistent, isolated and durable database transactions
- Journal storage \- records change sequentially in document-oriented format

### AWS Elastic Transcoder

Fully-managed video transcoding service, converting from one format to another for on-demand video (VOD) or streaming. Cannot be used via CloudFormation (AWS SDK / AWS CLI for automation only).  
To create a pipeline job, choose a preset (what to convert the video to) and a source and destination bucket.  
New version of Elastic Transcoder is called AWS Elemental MediaConvert.

### AWS Elemental Media Convert

Fully-managed video transcoding service \- new more comprehensive version of Elastic Transcoder with more processing options:

- Video correction
- Input filtering
- Cropping
- Overlaying
- Audio track / caption insert
- Etc.

Define job, input and outputs. Pulls from source bucket, transcodes and places in destination bucket.

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

### AWS Service Catalog

Create and manage catalogs of products approved for use on AWS. Alternative to granting direct access to AWS resources via the AWS console. Provides:

- Standardisation
- Self-service discovery and launch
- Fine-grained access control
- Entensibility and version control

Administrative side:

- Administrative User \- manages the catalog
- Portfolio \- container for sets of:
  - Product \- CloudFormation template
    - Can associate budget
    - Product must be removed from portfolio and not provisioned, to delete
  - Permissions \- who can view / launch products
    - Groups
    - Roles
    - Users
  - Constraints \- rules applied on provisioned products
    - Launch \- use IAM role instead of user credentials
    - Notifications \- send product notifications to a stack
    - Template \- limit the options when launching a product (e.g. limit hardware that can be provisioned to t2.micro)
    - StackSet \- configure product deployment across accounts and regions
    - TagUpdate \- allow or disallow end users updating tags on associated resources

Consumer side:

- End user \- uses the catalog
- Catalog \- user-friendly console to view / launch products
  - Products \- as above, viewable and launchable
  - Provisioned products \- active CloudFormation stacks
    - Service actions \- SSM documents to perform tasks on the stack

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

## AWS Lambda

Serverless functions as services, allowing code to run without provisioning or managing servers. Costs for compute time consumed \- no cost when not running. Lambda executes only when needed and scales automatically to 1000 functionals concurrently in seconds. Supports various runtimes including custom environments. Commonly used to glue services together without having constantly running servers. Lambda can be triggered by many services including external.  
Invocation types:

- Sync \- wait for a response (e.g. HTTP response)
- Async \- immediately returns to async AWS services such as:
  - SNS topic
  - SQS queue
  - Lambda function
  - EventBridge bus

Can configure:

- Timeout \- default=3s, min=1s, max=15m
- Storage \- default=512MB, min=512MB, max=10GB

Function Versions  
Can manage deployment of Lambda functions (e.g. \$LATEST). Each version has its own ARN. The ARN of the function without the version suffix is the unqualified version (e.g. arn:aws:lambda:aws-region:acc-id:function:helloworld); the ARN with the version suffix is the qualified version (e.g. arn:aws:lambda:aws-region:acc-id:function:helloworld:\$LATEST). These two are effectively identical because unqualified ARNs point to the latest version. Unqualified ARNs cannot use Aliases \- friendlier names to specific versions when referring to them programmatically.

Layers  
ZIP archives containing libraries, custom runtimes or other dependencies, allowing them to be used without including in the deployment package.  
Limits:

- Up to 5 layers per function
- Total unzipped deployment package up to 250MB

Instruction Sets  
Two available instruction architectures for lambda: arm64 and x86_64. ARM is more efficient due to having smaller instruction sets. All AL2 runtimes support both architectures.

Lambda Runtimes  
Preconfigured environments to run specific programming languages. Useful because they don’t require you to configure a container or OS config. Fully-managed and secure. Older runtimes are deprecated and upgrades are required.

OS-Only Runtimes  
Used when there is no preinstalled programming language / specific libraries installed when you want to compile.  
Cases:

- Native ahead-of-time (AOT) compilation
  - Languages like Go, Rust, C++ compile natively to an executable binary which doesn’t require a dedicated language runtime
  - Must include a runtime interface client in the binary
  - Must compile binary for Linux env for same instruction set architecture
- Third-party runtimes
  - Community or company-created runtimes for languages not supported normally
- Custom runtimes
  - Build your own

OS-only runtimes: Amazon Linux 2 \= provided.al2, Amazon Linux 2023 \= provided.al2023.

Deployment Packages  
Package containing the function code Lambda will deploy.  
Types:

- ZIP archive \- zip contents of code and additional libraries
  - AWS uploads contents to AWS-managed S3 bucket or you can upload to S2 bucket and reference address in Lambda
    - ZIP \> 50MB must be uploaded second way
  - Rely on Lambda runtimes (limited to these envs)
- Container image \- build a container
  - Create Dockerfile
  - Build image
  - Push image to registry (e.g. Amazon Elastic Container Registry \- ECR)
  - Reference container image to Lambda
  - Do not need to specify runtime (you’re doing it in the image) but slower

## Step Functions

Coordinate multiple AWS services into serverless workflows by creating a state machine (abstract model moving between states based on conditions) with micro-services. Displayed in visual workflow. Automatically triggers, tracks and logs each step and retries when there are errors.  
State machine types:

- Standard \- general purpose (long workloads)
- Express \- for streaming data (short workloads)

Steps can be executed in parallel. States are configured through Amazon States Language (JSON).

### Use cases:

- Manage a Batch job or Fargate container
  - Run, then notify SNS on success or failure
- Transfer data records
  - Load DynamoDB
  - Add each item to SQS queue
  - Remove items from DB as they are added to queue
  - Report success when done

\[Read through use cases / examples in documentation for exam\]

### States

**Pass states** pass their input to output without performing real work (mocks). Useful when constructing / debugging state machines.  
Made up of:

- Parameters \- key-value pairs passed as input
  - E.g. “ship”: “enterprise”
- Result \- virtual task to be passed to next state
  - E.g. “government”: “federation:
- ResultPath \- where to place the output of the mock
  - E.g. “\$.politics”
- Output \- what it produces
  - E.g. { “ship”: “enterprise”, “politics”: { “government”: “federation”}}

**Task states** are units of work performed by a state machine. Work performed by:

- Calling an AWS Lambda function
  - Pass the lambda ARN
- Passing parameters to API actions of other services
  - Pass ARN & parameters (varying per service)
  - Available services:
    - Lambda
    - AWS Batch
    - DynamoDB
    - ECS/Fargate
    - SNS
    - SQS
    - SageMaker
    - EMR
    - StepFunctions
- Using an activity
  - Workflow waits for activity worker to poll for the task
  - Allows task to be completed by a worker hosted anywhere

**Choice states** add branching logic to a state machine. Use logical conditions with NOT, AND operators etc to construct branches. Default branch can be given.

**Wait states** delay the state machine from continuing for a specified time. Can wait defined time, or wait until.

**Succeed states** stop execution successfully. Useful for branches that do nothing but stop execution.

**Failure states** stop the execution of the state machine while marking as a failure. They can provide types, causes and errors.

**Parallel states** execute steps in parallel and do not advance the state machine until they are all complete.

**Map states** iterate over an array, completing the step for each item. Good for inserting records to database etc.

### Inputs & Outputs

Step functions take JSON event data as input and produce JSON as output. The JSON payload can be manipulated using:

- InputPath \- select a portion of the state input
- Parameters \- create a collection of key-value pairs that are passed as input
  - When using JSONPath for parameters, append “.\$” to the key name
- ResultsSelector \- change a state’s result before the ResultPath is applied (opposite to parameters essentially)
- ResultPath \- determines what should be output \- the input, task output or a combination
- OutputPath \- select a portion of the state output to pass to the next state

Amazon States Language uses JSONPath syntax to identify components. This is a query language for JSON using an expression starting with the root (\$) and using dot or bracket notation to select children.

## AWS Compute Optimiser

Analyses current configuration of AWS compute resources and their utilisation metrics from CloudWatch over the last 14 days and makes suggestions.  
Can make recommendations for:

- EC2 instances
- Auto-Scaling Groups (ASGs)
- EBS volumes
- Lambda functions
- ECS services on Fargate
- SQL server licenses

## Elastic Beanstalk (EB)

Platform as a Service (PaaS) allowing customer to develop, run and manage applications without being hands-on with the underlying infrastructure. Like the Heroku of AWS. Not recommended for Production applications. Powered by CloudFormation. Pick a platform (language) \- can even just be dockerised containers. Has its own CLI.  
Environments:

- Web \- run/serve a web application
  - Load-Balanced env \- designed to scale
    - Creates an ASG
    - Creates an ELB
  - Single-instance env
    - Creates an ASG but desired capacity set to 1
    - No ELB
    - Creates Elastic IP
    - Public IP has to be used to route traffic to server
- Worker \- perform computational work
  - Creates and ASG
  - Creates an SQS queue
  - Installs SQS Daemon on EC2 instances
  - Creates CloudWatch alarm to dynamically scale instances based on health

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

## ElastiCache

Fully-managed in-memory datastore for Memcached or Redis, intended to cache data or HTML fragments to improve response times down in the 10s-100s of milliseconds.  
ElastiCache:

- Only accessible by resources in same VPC
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

Redis-compatible in-memory database for fast performance. Main difference from ElastiCache is MemoryDB has persistence guarantees making it **suitable as a primary database**. Writes are slower than ElastiCache (milliseconds vs microseconds) as a trade-off. Billed per usage.

## AWS CloudTrail

Monitors API calls and actions performed on an AWS account \- enabling governance, compliance and auditing of AWS accounts. Provides who, what, when and where of all actions. On by default and logs are preserved for the last 90 days via Event History. If longer than 90 days needed, create a Trail which outputs to S3 and requires Amazon Athena to analyse efficiently (CloudTrail Lake uses Athena under the hood). Commonly forwarded to CloudWatch logs.

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

- Single node \- 160GB
- Multi-node
  - Leader node \- manages client connections and receiving queries
  - Compute nodes \- store data and perform queries
    - Up to 32 by default but service request takes up to 128 (each 160GB)

Node types:

- Dense compute (dc) \- best for performance, less storage
- Dense storage (ds) \- best for storage, lower performance
- Start at large \- no smaller

Redshift uses Massively Parallel Processing (MPP), which automatically distributes data and query loads across all nodes.  
Backups enabled by default with 1 day retention (can be set up to 35 days). Redshift always maintains at least 3 copies of data:

- Original copy
- Replica on compute nodes
- Backup copy in S3

Can asynchronously replicate snapshots to S3 in different regions.  
Billed for compute node hours (not leader node), backup storage and data transfers.  
Data encrypted in transit and at rest. Via KMS or CloudHSM.  
Redshift is **single-AZ.** Would need to run clone in different AZ manually.

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

## ML Services

### Amazon CodeGuru

Machine-learning code analysis service comprised of:

- CodeGuru Security \- detect, track and fix code security issues
  - Code security analytics scan
  - Code quality analytics scan
  - Secrets detection scan
- CodeGuru Profiler \- find and fix inefficiencies in code
- CodeGuru Reviewer \- associate a repo and get continuous code change recommendations

Supports various languages.

### Amazon Comprehend

Natural Language Processor (NLP) service to find relationships between text in order to produce insights \- e.g. looking at data like customer emails, support tickets etc.  
Can analyse text and extract:

- Entities \- (e.g. person, organisation, location)
- Key phrases \- text appearing important (e.g. You need to **pay** the amount of **\$200** by **September 31st**.)
- Language \- (e.g. confidence of the language being spoken)
- PII
- Sentiment \- attitude towards the text (e.g. 0.20 negative)
- Targeted sentiment \- specific words and their attitude (e.g. awful 1.0 negative)
- Syntax \- identify parts of a language (e.g. hello proper noun)
- Custom models \- upload training data to analyse and extra custom text
  - Amazon Comprehend Flywheel \- automates the training of model versions for custom models

Comprehend is serverless and billed on size of requests in units e.g. 1 unit \= 100 characters. Real-time analysis can be performed via an endpoint (or custom endpoint for custom models). Analysis jobs allow for batch jobs. Provides confidence scores for each finding.

### Amazon Forecast

Time-series forecasting service for business outcomes such as product demand, resource needs or financial performance. Need to upload dataset to S3 with historical data and optional additional metadata.  
Workflow:

- Create data set group / data import job
  - Define schema
  - Register task
- Create predictor / get accurate metrics
  - ELT job evaluates the model
    - Choose predefined backtest
- Create forecast
  - Deploy the predictor
  - Retrained with full dataset
- Query / export forecast

Produced visual graph.

### Amazon Fraud Detector

Fully-managed fraud detection service identifying potentially fraudulent online activities e.g. online payment fraud, creation of fake accounts. Comes with predefined models against which you train your data:

- Online fraud insights \- optimised for fraud with little historical data (new account registration)
- Transaction fraud insights \- test for fraud cases where entity being evaluated might have history the model can use to improve prediction accuracy
- Account takeover insights \- for accounts compromised by phishing or other attack

Often used with step functions, Lambda, Kinesis etc for real-time fraud detection and alerts.  
Upload training dataset to S3.  
Components:

- Models \- (user defined)
- Thresholds \- (user defined)
- Scores \- numerical values representing estimated risk level (model generated)
- Rules \- interpret variable values during fraud prediction (user defined)
- Detector
- Outcomes \- define the fraud prediction result e.g. risk level and actions (user defined)
- Events
  - Entities \- who is performing the event (e.g. customer)
  - Labels \- classify an event as fraudulent or legitimate
  - Variables \- data points used in model (e.g. location, transaction amount)

### Amazon Kendra

Enterprise machine learning search engine service using natural language to suggest answers to questions instead of basic keyword matching. ‘Like interacting with a human’. Amazon Lex chatbot can be used as an interface to Kendra. Returns document results \- more like a librarian recommending books based on your question rather than them having knowledge to answer your question about any book..  
Components:

- Index \- table holding indexes of your documents to make them searchable
- Data source \- where documents are stored (e.g. S3, Sharepoint, Postgres)
  - Data source template schemas \- AWS provides around 40 schema templates for common AWS services or third-party cloud storage services
- Document addition API \- API to add documents directly to an index

Versions:

- Developer
  - 5 indexes with 5 data sources each
  - 10,000 documents / 3GB extracted text
  - 4,000 queries per day / 0.05 per second
  - 1 AZ
- Enterprise
  - 5 indexes with 50 data sources each
  - 10,000 documents / 3GB extracted text
  - 8,000 queries per day / 0.1 queries per second
  - 3 AZs

### Amazon Lex

Conversation interface service to build voice and text chatbots. Provides natural language understanding (NLU) and automatic speech recognition (ASR).  
Use:

- AWS-provided bot templates for common industries
- Transcripts to create a new bot
- Gen AI to build a bot by describing it
- Target languages and AWS-provided voices

Integrates with AWS Lambda to connect with other services.  
Amazon Lex Network of Bots is a feature to add multiple bots to a network and intelligently route the query to the appropriate bot.  
Components:

- Bot \- performs automated tasks, input to interact with conversational model
  - Version \- snapshot of bot model
    - Alias \- label/tag for version
  - Language \- target language(s) bot can converse in
- Intent \- action the user wants to perform
  - Sample Utterance \- text describing intent from user POV
    - “Can I order a pizza?”
    - “I want pizza”
    - “Hey there, any chance of a pizza?”
  - Slot \- inputs an intent requires from the user (can be 0\)
    - Slot type \- data type of the slot
      - Custom enumeration values (e.g. “Small”, “Medium”, “Large”)
      - Predefined data type (e.g. AMAZON.Number)

### Amazon Personalize

Real-time recommendations service. Used to make product recommendations to customers shopping on Amazon.  
Process:

1. Create **data set group**
2. Upload **data set** to data set group (**CSV** files)
   1. Provide multiple data sets
      1. User item interaction data (**required**)

- Core dataset used to train custom model
- Contains at least:
  - USER_ID
  - ITEM_ID
  - TIMESTAMP  
    2. User data (optional)
- Metadata about users to improve recommendation quality
- Contains at least USER_ID  
  3. Item data (optional)
- Metadata about the items to improve recommendation quality
- Contains at least:
  - ITEM_ID
  - CATEGORY_L1
  2. Need a **JSON schema mapping** for CSV files
  3. Reference the dataset location from an S3 object location

3. **Solutions** and **Recipes** allow fine tuning of models
   1. **Solutions** helps generate recommendations
   2. **Recipes** are predefined AWS algorithms
4. **Event trackers** \- Can track user events and feed into solution using Ingestion SDK
5. **Filters** \- Remove certain items from recommendations based on rules
6. Launch a **Campaign** to allow applications to get recommendations from solutions

### Amazon Polly

Text-to-speech service providing a spoken audio file in a synthesised voice in response to uploaded text.  
Engine types:

- Standard (\$) \- not as natural-sounding as other engines, but most cost-effective
- Long form (\$\$) \- sounds more natural than standard when reading longer text
- Neural (\$\$\$) \- most natural-sounding speech, supports newscaster and narration speaking styles

No standard speed \- variation between voices.  
Lexicon \- for specialised pronunciation

- Lexicon file (.xml, .pls) with up to 40,000 chars and 100 pronunciation rules

Speech Marks \- metadata describing the speech

- Where a word starts / ends
- Utilise Speech Synthesis Markup Language (SSML) (XLM-based)
- Integrate with Visme (third-party marketing material creation service)

### Amazon Rekognition

Image and video recognition service.  
Prebuilt use cases:

- Object detection
- Face detection
- Searching faces in connection
- People pathing
- Detecting personal protective equipment
- Recognising celebrities
- Moderating content
- Detecting text
- Detecting video segments
- Detecting face liveness

Also allows custom labels to identify specific objects, logos etc. specific to business needs.  
Image requirements:

- Jpg / png format
- Base64 encoding (automatically done by SDKs)

Can access images from S3 bucket.

Amazon Textract  
Service to extract text from scanned documents (OCR) to digitally extract data from paper forms.  
Capabilities:

- OCR documents
  - Retain layout coordinates
  - Convert to a table
  - Detect forms
  - Query against the OCR data
  - Detect signatures
- OCR expenses (e.g. receipts)
- Analyse IDs (e.g. drivers license / passport)
- Analyse lending (mortgage documents)
- Custom queries \- train your own models with uploaded samples

Amazon Translate  
Neural machine learning text translation service.  
Processing modes:

- Real-time translation
- Async batch processing

Provide:

- Text
- Source language
- Target language

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

## Open API

OpenAPI specification (OAS) defines a standard, language-agnostic interface to RESTful APIs allowing humans and computers to discover and understand the capabilities of a service without having access to the source code, documentation or network traffic inspection.  
Swagger and OpenAPI used to be the same thing but as of OpenAPI v3, OpenAPI \= specification, Swagger \= tools to implement specification.  
OpenAPI can be JSON or YAML.  
AWS extends the OpenAPI definitions, allowing definition of AWS API Gateway-specific features in an OpenAPI file (e.g. policies, CORS).

## Amazon API Gateway

API Gateways are programs sitting between a single entry point and multiple backends, allowing for throttling, logging, routing logic or formatting of requests & responses.  
Amazon API Gateway allows the creation of secure APIs at any scale, acting as a front door for applications to access data, business logic or functionality from backend services.  
Can have caching or logs and lead to lambda functions, elastic containers, dbs etc.  
Versions (not incremental \- different use cases):

- REST API (v1)
  - Complete control over request and response
  - Most feature-rich
  - Higher costs
  - Public and private API options
- HTTP API (v2)
  - Low latency
  - Simpler feature set
  - Low costs
  - Only public APIs
- WebSockets API
  - Persistent connections for real-time use cases

For both REST and HTTP APIs you can import an Open API 3 file when creating the API.

### REST vs HTTP

Learn for exam.

#### Endpoint Types

| Type           | REST | HTTP |
| -------------- | ---- | ---- |
| Regional       | Yes  | Yes  |
| Edge-optimised | Yes  | No   |
| Private        | Yes  | No   |

#### Security

| Type                          | REST | HTTP |
| ----------------------------- | ---- | ---- |
| Mutual TLS authentication     | Yes  | Yes  |
| Certificates for backend auth | Yes  | No   |
| AWS WAF                       | Yes  | No   |

#### Authorisation

| Type                                 | REST                     | HTTP |
| ------------------------------------ | ------------------------ | ---- |
| Resource policies                    | Yes                      | No   |
| IAM                                  | Yes                      | Yes  |
| Amazon Cognito                       | Yes                      | Yes  |
| Custom auth with AWS Lambda function | Yes                      | Yes  |
| JWT                                  | No (possible via Lambda) | Yes  |

#### API Management

| Type                        | REST | HTTP |
| --------------------------- | ---- | ---- |
| Custom domains              | Yes  | No   |
| API keys                    | Yes  | Yes  |
| Per-client rate limiting    | Yes  | Yes  |
| Per-client usage throttling | Yes  | Yes  |

#### Development

| Type                    | REST | HTTP          |
| :---------------------- | :--- | :------------ |
| Automatic deploys       | No   | Yes (default) |
| User-controlled deploys | Yes  | Yes           |
| CORS config             | Yes  | Yes           |
| Test invocations        | Yes  | No            |
| Caching                 | Yes  | No            |
| Custom gateway response | Yes  | No            |
| Canary release deploys  | Yes  | No            |
| Request validation      | Yes  | No            |
| Request body transform  | Yes  | No            |
| Request param transform | Yes  | Yes           |

#### Monitoring

| Type                 | REST | HTTP |
| -------------------- | ---- | ---- |
| CloudWatch metrics   | Yes  | Yes  |
| CloudWatch logs      | Yes  | Yes  |
| Amazon Data Firehose | Yes  | No   |
| Execution logs       | Yes  | No   |
| AWS X-ray tracing    | Yes  | No   |

#### Integrations

| Type                                | REST | HTTP                   |
| ----------------------------------- | ---- | ---------------------- |
| Mock integrations                   | Yes  | No                     |
| Public HTTP endpoints               | Yes  | Yes                    |
| AWS services                        | Yes  | Yes (limited services) |
| AWS Lambda functions                | Yes  | Yes                    |
| Private integrations with NLB       | Yes  | Yes                    |
| Private integrations with ALB       | No   | Yes                    |
| Private integrations with Cloud Map | No   | Yes                    |

### REST

Components:

- Stages \- versions of the API
  - E.g. prod, staging, dev
  - Must be deployed to be accessible
- API \- container for multiple resources
- Resources \- represent endpoints
  - Resources can be nested within other resources
    - E.g. /users/show
- Methods \- individual methods for specific endpoints
  - E.g. GET, PUT
  - Customise requests and responses
  - Flow:

1. Method request
2. Integration request
3. Integration
4. Integration response
5. Method response

- Integration \- service that will be called
  - Lambda functions
  - HTTP, Mock
  - AWS Service
  - VPC link

### HTTP

Components:

- Stages \- versions of the API
  - E.g. prod, staging, dev, \$default
  - Auto-deployed to default
- API \- container for multiple routes
- Routes \- represent an endpoint
  - Can be nested within other routes
  - Choose one method and one endpoint
    - E.g. PUT /hello
    - One route for each method-endpoint combination you want to support
- Integration \- service that will be called
  - Lambda functions
  - HTTP, Mock
  - AWS service
    - Limited to a handful
      - EventBridge
      - SQS
      - AppConfig
      - Kinesis Data Streams
      - Step Functions
  - VPC link

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
  - Provided by default via databases DNS endpoint

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
  - Multi AZ instances deployment is for RDS Instances
  - Multi AZ cluster deployment is for Aurora Clusters
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
- Up to 5 replicas for MySQL, MariaDB & PostgreSQL dbs
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

|                        | Multi-AZ Deployments                       | Read Replicas                                 |
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

Aurora Serverless Provisioned is the default compute configuration for Aurora. Aurora db cluster contains:

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
- Does not scale to 0 \- must maintain at least 0.5 ACUs
  - Aurora Capacity Units (ACUs) determine cost vs capacity
  - 1 ACU is about 2GiB memory, CPU and networking

Serverless vs. Provisioned

| Attribute          | Serverless                                                   | Provisioned                                      |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------ |
| Scaling            | Fine-grained, almost instant scaling                         | Manual scaling. Requires planning & downtime     |
| Capacity range     | 0.5-128 ACUs \- flexible                                     | Fixed, based on instance size chosen             |
| Scaling speed      | Seconds                                                      | N/A (manual intervention required)               |
| Read/Write scaling | Independent                                                  | Depends on instance type and read replica config |
| Compatibility      | Broader version support                                      | Wide version support, depending on instance type |
| Use cases          | Highly variable workloads needing frequent/immediate scaling | Stable/predictable workloads                     |
| Billing            | ACUs per second                                              | Instance hours & storage                         |
| Start/stop         | Responsive                                                   | Manual                                           |
| Maintenance        | Minimal downtime                                             | Scheduled maintenance windows                    |

Global Database:

- An Aurora database spanning multiple regions for global low-latency and high availability
- Has up to 5 secondary db clusters in different regions
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

## Elastic Container Registry (ECR)

Fully-manage Docker registry making it easy to store, manage and deploy Docker container or Open Container Initiative(OCI) images.  
Can deploy from ECR:

- Via the Elastic Container Service (ECS)
- Via Fargate
- Via Elastic Kubernetes Service (EKS)
- On-premise

With ECR you can:

- Control access
  - Private register access via Register policy
  - Private repo access via Repo policy
- Scan images on push to identify software vulnerabilities
- Have cross-account and cross-region images using private image replication
- Create a Pull Through Cache to sync contents of upstream registry
- Manage automation of cleaning up container images using ECR Lifecycle
- Sign images via AWS Signer to verify trusted developers
- Tag with mutable or immutable tags
  - Immutability is best practice for rollback etc.
- Have public or private registries

ECR encrypts repo images at rest.  
Composition:

- Registry \- contains multiple repositories
- Repository \- contains multiple images
- Image \- packaged, ready-to-run software file containing everything it needs (code, libraries, settings)
  - Can have multiple tags
- Tag \- points to specific image versions
  - E.g. 1.0, latest

### Elastic Container Service (ECS)

Container orchestration service to run multiple containers across multiple EC2 machines managed in a cluster. Fargate is marketed separately but is basically fully-managed ECS.  
Terms:

- Cluster \- multiple EC2 instances housing the docker containers
- Task definition \- JSON file defining the config of up to 10 containers to run, made up of:
  - Family \- a way to group similar task definition
    - This is how its versioning works
  - Execution role \- role used to manage container
  - Task role \- role used by compute running container
  - Network mode
    - Host \- basic mode, connect directly to the host machine
    - Bridge \- isolate between containers but they can still communicate
    - AWSVPC \- creates an ENI in your VPC with a private IP address
      - Fargate can only use this mode
    - None \- disable networking
  - CPU & memory \- how much memory and compute
  - Requires compatibilities \- EC2, Fargate, External
  - Container definition \- defines connection of containers to be provisioned on compute
    - Name \- name of the container
    - Image \- URI to the container image (ECR, DockerHub)
    - Essential \- must be one essential container, if this fails all containers fail
    - Health check \- perform a health check
    - Port mappings \- map the guest to host ports (mode dependent)
      - Bridge mode
        - Container (guest) port maps to host port
        - Ports can be different for each
      - Host mode
        - Container (guest) port directly maps to host’s same port
        - Do not need to define host port
      - AWSVPC
        - Gives container its own network interface with direct public IP
        - Guest and host port will be the same
        - Do not need to define host port
    - Log configuration \- write logs to AWS CloudWatch
      - Log driver \- tells container where to log
        - ‘awslogs’ will log to CloudWatch Logs
          - Blocking by default, may want nonblock
          - Or use AWS Firelens
            - Runs in sidecar in same container as task to avoid backpressure
        - Third party log drivers available
        - Many more available for EC2 than Fargate
    - Environment \- env vars you want to set for the container
    - Secrets \- secrets from Secrets Manager or SSM Parameter Store
- Task \- launches containers
  - Do not remain running once workload is complete
- Service \- ensures tasks remain running \- e.g. web apps
- Container agent \- binary on each EC2 instance which monitors, starts and stops tasks
- EC2 controller / scheduler \- responsible for scheduling the redeployment and placement of containers
  - Replaces unhealthy containers
  - Can create own or use third-party ones

### Fargate

Serverless container orchestration service where AWS manages the underlying EC2 servers so you don’t need to scale or upgrade.  
Details:

- Can create an empty ECS cluster and then launch tasks as Fargate
- Charged for at least one minute, then by the second
- Charged for duration and consumption
- Must use awslogs networking mode
- Will have an ENI in the VPC per task group
- Must use IP addresses when using ELB to point to Fargate
  - Fargate tasks do not have hostnames

Configuring:

- Define memory and vCPU in task definition
- Add containers and allocate memory and vCPU for each
  - Memory min and max increase as vCPUs increase
- When running task, choose which VPC and subnet run it
- Apply Security Groups to tasks
- Apply IAM roles to tasks
- Just like ECS

### Executor vs. Task Role

ECS Execution Role is the role used to prepare / maintain the container. Common permissions:

- Access to Secrets Manager / SSM Parameter Store
- Access to download private image from ECR
- Full access to CloudWatch Logs

ECS Task Role is the role used by the running compute of the container. Common permissions:

- Access to SSM Messages for the ECS Exec
- Full access to CloudWatch Logs so container can log
- XRay Daemon Write Access so Xray can be used for traceability

### Capacity Providers

Manage the scaling of infrastructure for tasks within clusters. Each cluster can have **one or more** capacity providers and an optional **capacity provider strategy**.  
Fargate has predefined capacity providers:

- FARGATE
- FARGATE SPOT
- Can create custom ones too

For ECS:

- Create an Auto Scaling Group and associate with custom capacity provider
- Attach custom capacity provider to ECS EC2

### Task Lifecycle

States:

1. Provisioning \- additional steps before the task is launched
   1. E.g. launching and attaching ENIs
2. Pending \- waiting on the container agent to take further action
3. Activating \- perform additional steps after the task is launched but not before it is running
4. Running \- task successfully operational
5. Deactivating \- perform additional steps before the task is stopped
6. Stopping \- waiting on the container agent to take further action
7. Deprovisioning \- additional steps after the task has stopped
   1. E.g. detaching and deleting ENIs
8. Stopped \- task successfully non-operational
9. Deleted \- task destroyed

### ECS Exec

Allows direct interaction with containers without needing to interact with the host container OS, open inbound ports, or manage SSH keys.

- Works with both ECS EC2 containers and ECS Fargate containers
- Commands are run as root
- Commands cannot be executed via the Management Console \- must use a terminal
- Session has an idle timeout of 20 minutes
- Must be turned on at time of task launch
  - Also must be enabled in cluster

Prerequisites:

- AWS CLI installed
- Sessions Manager Plugin installed
- Task role must have permission
- Must meet ECS/Fargate version requirements

Recommended to set initProcessEnabled for Linux to avoid zombie SSM agent children.

### ECS Service Connect

Makes it easy to setup a service mesh for service-to-service communication. Evolution of App Mesh, abstracting much of the configuration between App Mesh, Cloud Map and ELB.  
Features:

- Service discovery
- Consistent approach to handling service-to-service communications
- Encrypts data in transit between services using TLS
- Telemetry data in the ECS Console and CloudWatch
- Traffic health checks
- Automatic retries
- Rolling deployments

Deploys a sidecar proxy container e.g. Envoy. Can use the service discovery name to easily talk to other services.  
Creating:

- When creating a cluster, define a ServiceConnectDefaults
  - This creates a CloudMap Namespace
  - When creating your service, configure it for Service Connect
    - Providing the Namespace, Discover Name and Port Name

ECS Optimised AMI  
AMIs preconfigured with the requirements and recommendations to run your container workloads. When launching an EC2 instance via ECS EC2 Management Console, it will automatically use an EC2 Optimize AMI by default.  
Features:

- Comes with Docker installed
- Comes with ECS Container Agent installed
- OS-level optimised for containers
- Variant of ECS optimised with GPUs

AMI can be changed in the launch template.  
Battlerocket:

- Linux-based open-source OS purpose-built by AWS for running containers on virtual machines or bare metal hosts
- Does not include a package manager
- Software can only be run as containers
- Updates applied and can be rolled back in a single step
  - Reduces likelihood of update errors
- Do not support:
  - ECS Anywhere
  - Service Connect
  - Amazon EFS in encrypted mode / AWSVPC network mode

ECS Anywhere  
Allows registration of external VMs from on-premise network to ECS cluster.

- Costs \$0.01025 /h for each managed ECS Anywhere on-premise instance
- Can register an external instance to a single cluster
- External instances require ECS Anywhere IAM role to allow them to communicate with AWS APIs
- ECS Exec supported
- Not supported:
  - AWSVPC network mode
  - Service load balancing
  - Service discovery
  - ECS capacity providers
  - SELinux
  - EFS volumes
- Uses launch type EXTERNAL
- Can run on Windows but Windows license required
- To install:
  - Create SSM Activation pair
  - Download install script to machine
  - Run install script
    - Runs and starts ECS Service agent
    - Service agent managed via systemctl

## Elastic Kubernetes Service (EKS)

Managed service eliminating the need to install, operate and maintain your own Kubernetes control plane on AWS.

![][image2]

- Connect and manage cluster via KubeCTL
- Use ALB to route traffic to nodes via AWS ALB ingress controller
- Options for compute nodes:
  - EC2 instances
    - Managed Node groups
      - Auto scaling fully managed by AWS
    - Self-managed Node groups
      - Customers manage scaling using EC2 Auto Scaling Groups
    - Karpenter
      - Cloud-native open-source autoscaler
  - Fargate instances
  - External instances
    - On-premise
- Add ons:
  - Amazon VPC CNI plugin for Kubernetes
    - Enables pod networking within clusters
  - Core DNS
    - Enables service discovery within clusters
  - Kube-proxy
    - Enables service networking within clusters
  - Amazon EKS Pod Identity Agent
    - Grants AWS IAM permissions to pods through Kubernetes service accounts
  - Third-party add ons
- Register and connect conformant Kubernetes clusters to AWS and visualise in the EKS console using Amazon EKS Connector
  - Bring own K8 cluster to EKS
  - Install via heml in target clutter
- EKS CTL assists K8 cluster setup on AWS
  - Can deploy:
    - EC2-backed nodes
    - Fargate-backed nodes
    - To private cluster on AWS Outpost
  - Can be configured away from defaults using a config file

### EKS Distro (EKS-D)

Kubernetes distribution based on, and used by, EKS to create reliable and secure K8s (Kubernetes) clusters.  
Use cases:

- Hybrid developments \- consistency between AWS and on-premise
- Development & testing \- identical prod and dev environments
- AWS Services Extension \- AWS integration with on-premise setups

Supported installation method for EKS-D available with EKS Anywhere (EKS-A).

EKS Anywhere  
Deployment option for EKS to easily create and operate K8s clusters on-premise with own VMs or bare metal hosts.

- Deploys EKS Distro as the K8s distribution.
- Allows management of deployed clusters from AWS Management Console
- Admin machine required to run cluster lifecycle operations
  - Does not need to run continuously
  - Critical cluster artifacts saved to admin machine
    - E.g. Kubeconfig file, SSH keys, etc.
- Can be deployed to:
  - Docker (development clusters)
  - AWS Snowball Edge
  - VMWare vSphere
  - Apache CloudStack
  - More
- Open-source and free
  - Enterprise Subscriptions for 24/7 support are not
    - Tens of thousands per cluster per year

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

## Key Management Service (KMS)

Makes it easy to create, control and rotate encryption keys on AWS.  
Integrates with many services:

- RDS
- CodeCommit
- S3
- CodeDeploy
- Glacier
- SNS
- SQS
- DynamoDB
- EC2
- X-Ray
- ElastiCache
- Codebuild
- CloudTrail
  - To audit access history
- Many more

Most AWS services can just checkbox encryption and choose a KMS key \- simple.

- Managed ones free
- Custom ones have small ongoing cost

KMS is a multi-tenant Hardware Security Module (HSM).

- HSM \= hardware specialised for storing encryption keys
  - Designed to be tamper-proof
  - Stores keys in memory, so not written to disk
- Multi-tenant \= multiple customers using same hardware
  - Isolated virtually
  - CloudHSM is single-tenant HSM for stricter compliance
    - FIPS 140-2 level 3 complaint, vs level 2 for multi-tenant

CLI commands to know:

- aws kms …
  - create-key \- creates a unique customer managed customer master key in AWS account and region
  - encrypt \- encrypts plaintext into ciphertext using a customer master key
  - decrypt \- descrypts ciphertext that was encrypted by an AWS KMS CMK into plaintext
  - re-encrypt \- decrypts ciphertext and then re-encrypts it within KMS. Why?
    - Manually rotate a CMK
    - Change the CML that protects ciphertext
    - Change the encryption context of ciphertext
  - enable-key-rotation \- enables automatic rotation of the key material for the specified symmetric CMK
    - Must be symmetric
    - Cannot be performed on a CMK in a different account

### Customer Master Key (CMK)

Keys used to encrypt all other keys (data keys) on a system (envelope encryption). Master keys are stored in secure hardware. Customer master keys are logical representations of a master key.  
Includes:

- Encryption key material
- Decryption key material
- Metadata:
  - Key ID
  - Creation date
  - Description
  - Key state

KMS supports symmetric and asymmetric CMKs.

## CloudHSM

Single-tenant HSM as a service, automating hardware provisioning, software patching and backups. FIPS 140-2 level 3 compliant as single-tenant. Built on Open HSM industry standards to integrate with:

- PKCS\#11
- Java Cryptography Extensions (JCE)
- Microsoft CryptoNG (CNG) libraries

Can transfer keys to other HSM solutions, easy to migrate keys. Can configure AWS KMS to use CloudHSM cluster as a custom key store rather than using the default.  
More expensive than KMS.

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

## Amazon Certificate Manager (ACM)

Provision, manage and deploy public and private SSL/TLS certificates for use with AWS services.  
Certificate types:

- Public \- certificates provided by ACM
  - Free
- Private \- imported certificates
  - Costs (\$400/month\!)

Can be attached to:

- ELB (including ALB)
- CloudFront
- API Gateway
- Elastic Beanstalk (via ELB)

ACM can handle multiple subdomains and wildcard domains (e.g. example.com &\*.example.com).  
Example implementation:

- Traffic ingress from the internet, via Route53, through an ALB and into EC2 instances
- ACM certify at the ALB to terminate SSL at the load balancer
- This prevents you needing to install certificates on every instance
- Theoretically less secure, but unencrypted traffic is on own network at least

## Amazon Cognito

Customer Identity and Access Management (CIAM) system, providing authentication, authorisation and user management for web and mobile apps. Provides authentication to AWS services.  
Methods:

- Cognito User Pools
  - User directory with authentication to identity providers (IdP) to grant access to an app
- Cognito Identity Pools
  - Provide temporary credentials for users to access AWS services
- Cognito Sync
  - Syncs user data and preferences across all devices

![][image4]

Amazon Detective  
Analyse, investigate and quickly identify the root causes of security findings or suspicious activities.  
You can:

- Quickly identify trends for EC2 instances and IAM Principles for example
- See on a map where API calls are generally being made from
- See a summary list of how many times specific API calls have been made
- See how much groups of API calls have increased in volume
- Launch investigations on specific IAM principles to see if they are utilising specific tactics
  - Amazon Detective creates a behaviour graph when making determinations

## Amazon Directory Service

Provides multiple ways to use Microsoft Active Directory (AD). It lets you use Microsoft AD-aware or Lightweight Directory Access Protocol (LDAP) \-aware apps in the cloud.  
Offers:

- Simple AD (not in all regions) \- AD-compatible directory supporting very basic features
- AD Connector \- proxy service to connect to existing on-premise AD
- AWS Managed Microsoft AD \- full feature version managed by AWS
- Amazon Cognito \- integrate signup and sign-in into web apps

### Directory service

Maps names of network resources to their network addresses. Shared information infrastructure for locating, managing, administering and organising resources like:

- Volumes
- Folders
- Files
- Printers
- Users
- Groups
- Devices
- Telephone numbers
- Other objects

Critical component of a network operating system.  
Directory server (name server) \- server providing a directory service.  
Each resource on the network is considered an object and information about it is stored as a collection of attributes.  
Well-known directory services include:

- Domain name service (DNS)
- Microsoft Active Directory
  - Introduced in Windows 2000
  - Manage multiple on-premise infrastructure components and systems using a single identity per user
  - Forest of domain trees, each of which representing an organisational unit
- Etc.

### Lightweight Directory Access Protocol (LDAP)

Open, vendor-neutral industry standard application protocol for accessing and maintaining distributed directory information services over an IP network. Commonly used to provide a central place to store usernames and passwords. LDAP allows for same-sign on \- allowing users to have a single ID and password to access everything, which they need to enter each time.  
Example setup:

- On-premise active directory
- Communicates directly with an LDAP directory
- Which communicates with Google Cloud, Kubernetes and Jenkins

LDAP vs. SSO:

- Most SSO uses LDAP under the hood
- LDAP was not designed for web-apps
- Some systems only support integration with LDAP \- not SSO

## AWS Firewall Manager

Centrally configure and manage firewall rules across accounts and applications.  
Services that can be managed:

- AWS WAF (including classic)
- AWS Shield Advanced
- Security Groups
- Network Access Controls
- AWS Network Firewall
- Amazon Route53 Resolver DNS Firewall
- Third-party firewall services

Requirements:

- Account must be member of AWS Organisation
- Account must be AWS Firewall Manager admin
- Must have AWS Config enabled for accounts and regions
- Must have Resource Access Manager (RAM) enabled for specific services
  - E.g. AWS Network Firewall, Route53 resolver DNS firewall

Policy config varies based on targeted service.  
\$100 per month. Many policies can be achieved via AWS Config (often used under the hood with these types of services).

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

## Secrets Manager

Used to safely store and rotate secrets:

- Database credentials:
  - RDS
  - Redshift
  - DocumentDB
  - Other databases
- Key/Value:
  - Usually API keys

Enforces encryption at rest using KMS.  
Costs per secret per month and per 10,000 API calls.  
CloudTrail can monitor credentials access in case of audit.  
Database credentials can be automatically rotated:

- Intervals range from 30 days \- 365 days
- Performed by a Lambda function
- Can rotate password for the superuser or developer programmatically accessing the db

## AI Dev Tools

### Amazon Q

AI chatbot using multiple LLM models via Amazon Bedrock. Ask questions similar to ChatGPT or other generative AI chat services.  
Models:

- Amazon Q Business
  - Connect it to company data, information and systems
  - More than 40 built-in connectors
- Amazon Q Developer
  - Coding, testing and upgrading to troubleshooting and optimising AWS resources
  - Integrated into:
    - AWS Managed Console
    - VSCode via AWS toolkit
    - Cloud9
    - AWS Lambda Code Editor
    - Slack
    - More
- Amazon Q for Amazon QuickSight
  - Ask questions about BI data within QuickSight
  - Generative BI capabilities to:
    - Quickly build compelling visuals
    - Summarise insights
    - Answer data questions
    - Build data stories using natural language
- Amazon Q for Amazon Connect
  - Real-time conversation with the customer along with relevant company content
- Amazon Q for AWS Supply Chain
  - Get intelligent answers about what is happening in a supply chain

### Amazon CodeWhisperer

Real-time coding companion \- suggesting code as you’re writing.  
Provides:

- In-line code suggestions
- Public code filter and reference tracking
- Command line integration
- Amazon Q chat in IDE
- Security vulnerability scanning

Tiers:

- Individual
  - Authenticate via Builder ID
  - 50 users / month
- Professional
  - Authenticate via IAM Identity Centre
  - 500 users / month
  - Additional features:
    - Customise for organisations
    - Organisational license management
    - Organisational policy management
    - Amazon Q feature development
    - Amazon Q code transformation

Integrates with:

- AWS Glue Studio Notebook
- JetBrains (e.g. IntelliJ IDEA)
- JupyterLabe
- Amazon SageMaker Studio
- Terminal, shell & command-line
- VSCode
- Visual Studio

In the AWS Toolkit.

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

![][image5]

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

## AWS Shield

DDoS (Distributed Denial of Service) attacks are attempts to disrupt normal traffic by flooding a server with large amounts of fake traffic.

AWS Shield is a managed DDoS protection service that safeguards applications running on AWS. When you route traffic through Route53 or CloudFront, you are using AWS Shield Standard.  
Protects against attacks on layers:

- 3 \- Network layer
- 4 \- Transport layer
- 7 \- Application layer
  - Via integration with AWS Web Application Firewall (WAF)

TIers:

- Shield Standard
  - Free
  - Access to tools and best practices to build a DDoS-resilient architecture
  - Automatically available on all AWS services
- Shield Advanced
  - Costs (\$3000 / year)
  - Available on:
    - Amazon Route53
    - Amazon CloudFront
    - Elastic Load Balancing (ELBs)
    - AWS Global Accelerator
    - Elastic IP
      - Amazon EC1
      - Network Load Balancer
  - Notable features:
    - Visibility and reporting on layers 3, 4, & 7
    - Access to team and support
    - DDoS cost protection
    - Comes with SLA

## AWS Web Application Firewall (WAF)

Protects web applications from common web exploits. Write custom rules to allow or deny traffic based on the contents of HTTP requests. Otherwise, use a ruleset from a trusted AWS Security Partner from the AWS WAF Rules Marketplace.  
Can be attached to:

- CloudFront
- Application Load Balancers

Protect web apps from attacks covered in the OWASP top 10 most dangerous attacks:

1. Injection
2. Broken Authentication
3. Sensitive data exposure
4. XML external entities (XXE)
5. Broken access control
6. Security misconfigurations
7. Cross site scripting (XSS)
8. Insecure deserialisation
9. Using components with known vulnerabilities
10. Insufficient logging and monitoring

## Amazon Guard Duty

Threat detection system acting as both an intrusion detection system (IDS) and intrusion protection system (IPS). It continuously scans for malicious or suspicious activity using machine learning to analyse:

- CloudTrail logs
- VPC Flow logs
- DNS logs

It will alert you to findings and can automate an incident response via CloudWatch Eventbridge or third-party services. You can investigate an issue with Amazon Detective.

## AWS Personal Health Dashboard

Provides alerts and guidance for AWS events that might affect your environment. Shows recent events to help you manage active events and shows proactive notifications enabling planning for scheduled activities. Use alerts to be notified about changes that can affect AWS resources and follow guidance to resolve issues.

Not to be confused with the Service Health Dashboard, which shows general health of AWS services divided by high level region (North America, Europe, etc.). An icon and details column indicate the status of each service.

## AWS Artifact

Self-service portal for on-demand access to AWS compliance reports. Choose compliance type and download pdf. Explains AWS responsibilities and customer responsibilities to ensure standard is being met.

## Storage Gateway

Connects on-premise software applications to cloud-based storage.

- File Gateway \- run a gateway within on-premise environment so you can interact through SMB or NFS file-system protocol
  - Amazon S3 \- store data in S3
    - Gateway can be deployed on AMI in Amazon EC2
  - FSx \- store data in Windows File Server
    - Gateway can be deployed on AMI in Amazon EC2
- Volume Gateway \- mount S3 as a local drive using Internet Small Computer Systems Interface (iSCSI) protocol
  - Cached volumes \- primarily stored on S3, cached locally
    - Minimizes need to scale on-premise infrastructure while providing applications with low-latency data access
    - 1GB-32GB
    - Hosting:
      - Deploy as VM appliance
      - Deploy as hardware appliance
      - Deploy to EC2 instance
  - Stored volumes \- stored locally and entire set of data backed up to S3 asynchronously
    - Data written to volumes can be asynchronously backed up as snapshots
      - Stored in cloud as EBS snapshots
      - Incremental backups (just what’s changed)
      - Compressed
      - 1GB-16TB
    - Hosting:
      - Deploy as VM appliance
      - Deploy as hardware appliance
      - Deploy to EC2 instance
- Tape Gateway \- stores files on Virtual Library Tapes (VTLs) for backing up files on cost-effective long term storage
  - Virtual tape cartridges store data
  - Readability of 30 years
  - Pre-configured with media changer and tape drives available to existing client backup applications as iSCI devices
  - Hosting:
    - Deploy as VM appliance
    - Deploy as hardware appliance
    - Deploy to EC2 instance
