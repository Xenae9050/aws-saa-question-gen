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
    - IPv6 \= http\://**\[fd00:ec2::254\]**/latest/meta-data/
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
  - No reboot is required to attach a role to a running or stopped instance
  - Changing roles **not** instant (eventual consistency)
    - Prefer **replacing** the instance profile; removing a role can take up to an hour to take effect
- Are automatically created when selecting an IAM role during EC2 instance launch
  - Not easily viewable from the console

## Instance Lifecycle

**Actions**:

- Launch \- create and start instance based on AMI
- Stop \- turn off but not delete instance
- Start \- turn on a previously stopped instance
- Terminate \- delete instance
- Hibernate \- saves RAM contents to the (encrypted) EBS root volume then stops the instance. On start, RAM is reloaded and processes resume (fast warm-up). Must be enabled at launch; no instance charge while hibernated, but EBS storage is still charged
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
  - **Other regions** \= \[ec2-instance-id\].region.compute.internal (e.g. i-0123456789abcdef.ca-central-1.compute.internal)

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
  - Max of **7** partitions per AZ; can be **multi-AZ**
  - Ideal for large distributed / replicated workloads (e.g. Kafka, Cassandra)
- **Spread**
  - Each instance has its own underlying hardware (rack)
  - Ideal for separate critical instances
  - Max of **7** running instances per AZ
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

- Amazon Linux 1 (**AL1**) \- end of life 2023-12-31
- Amazon Linux 2 (**AL2**) \- end of support 2026-06-30
- **Amazon Linux 2023** (AL2023)
  - Contains many packages built in
  - Python 3 default
  - Security Enhanced Linux (SELinux) enabled by default (permissive mode)
  - OpenSSL 3
  - Components sourced from multiple Fedora versions and others (e.g. CentOS 9 Stream)
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
- Reserved \- best long term (up to 72% off)
  - Stead, predictable usages
  - Commit over 1 or 3 year term
    - Not auto-renew
    - Will go to on-demand after term
  - Can resell unused reserved instances (RIs) in the Reserved Instance Marketplace
  - Pricing \= Term x Class x Attributes x Payment option
    - Term \= length of commitment
    - Class \= type of RI
      - Standard \- up to 72% off
        - Can modify RI attributes:
          - Change AZ within same region
          - Change scope of zonal RI to regional or vice versa
          - Change instance size (same instance family and generation; Linux/UNIX only)
        - Cannot be exchanged
        - Can be bought or sold in RI marketplace
      - Convertible \- up to 66% off
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
    - Sell up to \$50,000 (lifetime limit, not annual)
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
- Must use awsvpc networking mode (‘awslogs’ is a log driver, a separate setting)
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

Makes it easy to setup a service mesh for service-to-service communication. Successor to App Mesh, abstracting much of the configuration between Cloud Map and ELB. AWS announced end of support for App Mesh on 30 September 2026.  
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

## AWS Compute Optimiser

Analyses current configuration of AWS compute resources and their utilisation metrics from CloudWatch over the last 14 days by default (up to 93 days with enhanced infrastructure metrics) and makes suggestions.  
Can make recommendations for:

- EC2 instances
- Auto-Scaling Groups (ASGs)
- EBS volumes
- Lambda functions
- ECS services on Fargate
- SQL server licenses
- RDS databases (including Aurora)
- Idle resources can also be identified
