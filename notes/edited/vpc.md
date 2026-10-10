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
**Do** cost money: NAT gateways, VPC endpoints (interface and Gateway Load Balancer; gateway endpoints are free), VPN gateways, Customer gateways, IPv4 addresses, Elastic IPs.  
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

## Shared VPCs

VPCs can be shared with other AWS **accounts** within the **same organisation** using AWS Resource Access Manager (**RAM**).  
This really just shares **subnets**. Can**not** share **default** subnets.  
You must **enable** sharing within the organisation via the RAM **API**.  
Must create: a **resource share** (what is being shared) and **shared principles** (who is being shared with) in **RAM**.  
Participants can only see and manage their **own** resources in a shared subnet - not other participants' or the owner's. The owner manages the subnets, route tables, NACLs and gateways.

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
- Carrier Gateway \- out to a telecom carrier network (AWS Wavelength Zones)
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
- Carrier Gateway \- Connect to a telecom carrier network (AWS Wavelength Zones)
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
- Charged at \$0.005/hour for every public IPv4 address (in use or idle), to encourage IPv6 use and releasing unused addresses back to the pool

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
**Two** bandwidth options: **Hosted** (via partner: 50Mbps-500Mbps, and up to 25Gbps from some partners), **Dedicated** (1, 10, 100, 400Gbps)  
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
  - Connects to only **S3** and **DynamoDB** (S3 and DynamoDB also support interface endpoints)
  - Specify the VPC to contain the endpoint and the desired service
  - **Un**idirectional
- **Gateway Load Balancer endpoints**
  - Use **PrivateLink**
  - Own endpoint type
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
- **Amazon Data Firehose**

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
  - **VPN tunnel** \- encrypted connection for data (specific pathway making up connection \- AWS usually provides **2 tunnels** for site-to-site for availability, in different AZs, up to 1.25Gbps each)
  - **Customer Gateway** (CGW) \- provides info to AWS about customer gateway **device**
    - Presents: BGP ASN (dynamic routing only), public IP address of the customer gateway device, and optionally a private certificate from AWS Private CA via ACM (certificate-based authentication)
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
  - IPv**6** traffic **not** supported on a Site-to-Site VPN on a **virtual private gateway** (must use **transit gateway**)
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
- **Zonal** NAT gateways sit in one AZ (in a public subnet) - deploy one **per AZ** for availability. A **regional** NAT gateway spans AZs automatically (no public subnet needed, public connectivity only)
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

**Nat Instances** are **legacy** deployments of NAT onto **individual EC2 instances** which have not been deprecated. These required the customer to handle scaling themselves. The AWS-provided NAT AMI is no longer supported; to use a NAT instance you must create your own NAT AMI (and disable source/destination check). AWS recommends NAT gateways instead.

## Bastion / Jumpbox

**Bastions / Jumpboxes** (same thing, two names) are **security-hardened virtual machines** providing secure access into **private subnets** via SSH or RCP. NAT gateways should **not** be used as Bastions as they are only intended for **outbound** access to the internet. AWS does **not** have its own Bastions, third party / community ones are available. System Manager’s **Sessions Manager** can replace the need for a Bastion. EC2 Instance Connect Endpoint is another option (no bastion or public IP needed).

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
- Each VPC attachment supports up to **100Gbps** per AZ (each direction)
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
- **Can** reference security groups in peered VPC in your security group rules (same Region only)

To VPC peer:

1. **Create** peering connection
2. **Accept** peering connection
3. **Add route** to route table for each VPC

## Network Address Usage

Network Address Usage (**NAU**) is a measure of resources in a virtual network.  
Limits:

- **VPC**s limited to **64**,000 NAUs (256,000 with an increase)
- **Peered VPCs** limited to **128**,000 NAUs (512,000 with increase) \- applies to **total** of peered VPCs (same-Region peering only)

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
