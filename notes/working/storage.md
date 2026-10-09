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
