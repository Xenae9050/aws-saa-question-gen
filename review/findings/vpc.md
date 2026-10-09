# Triage: VPC (notes/working/vpc.md)

## Summary

- Section reviewed: `notes/working/vpc.md` (475 lines; VPC basics, NACL/SG, route tables, gateways, IPs, Direct Connect, endpoints/PrivateLink, flow logs, VPN, NAT, bastion, Lattice, TGW, mirroring, Network Firewall, peering, NAU)
- Official exam guide checked: AWS Certified Solutions Architect – Associate (SAA-C03) Exam Guide, **Version 1.1** (https://d1.awsstatic.com/training-and-certification/docs-sa-assoc/AWS-Certified-Solutions-Architect-Associate_Exam-Guide.pdf; downloaded and text read). Date not extractable. In-scope list includes Client VPN, Direct Connect, PrivateLink, Site-to-Site VPN, Transit Gateway, VPC, Network Firewall. Task statements reference VPC architecture (SGs, route tables, NACLs, NAT gateways), VPN/Direct Connect/peering/TGW, NAT gateway types and NAT instance vs gateway cost.
- Live verification status: Live web access available. Official AWS docs pages were fetched and inspected for every CONFIRMED finding.
- Findings: 16 total. 13 CONFIRMED (ERROR 3, OUTDATED 5, INCOMPLETE 4, AMBIGUOUS 1) and 3 UNVERIFIED candidates (AMBIGUOUS 2, INCOMPLETE 1). 0 UNRESOLVED.
- Important limitations:
  - Several docs URLs (e.g. carrier gateway, public IPv4 pricing doc pages) redirected to index pages and could not be fetched; alternative official pages were used. The VPC pricing page's "Public IPv4 Address" tab did not render; the official AWS blog announcement was used for IPv4 charges.
  - Exam-guide date not available (only "Version 1.1").
  - Not every sentence was researched. Spot-checked with no finding: VPC/subnet/CIDR quotas, SG per-region (2,500 default)/rules (60)/SGs per ENI (5, up to 16), NAU limits and table, default-VPC components, NAT gateway public/private modes, VPC peering "no single point of failure", TGW 5,000 attachments, Flow Logs destinations.
  - VPC Lattice, Traffic Mirroring, Network Firewall, Client VPN, Gateway/GWLB endpoint directionality and the "shared subnets cannot include default subnets" claim were not examined in depth.
  - Typos (e.g. "subjects", "Firehouse", "NPM", "RCP", "BPG", "principles", "Wight") are STYLE and not reported.

## Findings

### FOUND-001 — VGW IPv4/IPv6 statement is reversed
- **Category:** ERROR
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** AWS VPN → Site-to-Site VPN → Limitations
- **Original claim:** "IPv**4** traffic **not** supported on **virtual private gateway** (must use **transit gateway**)"
- **Issue:** It is IPv6 that a VPN on a virtual private gateway does not support. The note's own Features bullet ("Support for IPv6 traffic using a transit gateway") is consistent with the correct version.
- **Official evidence:** "A Site-to-Site VPN connection on a virtual private gateway does not support IPv6 traffic. However, we support IPv6 traffic routed through a virtual private gateway to a Direct Connect connection."
- **Source:** Example routing options – Amazon VPC User Guide, https://docs.aws.amazon.com/vpc/latest/userguide/route-table-options.html (section "Routing to a virtual private gateway")
- **Assessment:** Direct contradiction of the note.
- **Suggested correction:** "IPv**6** traffic **not** supported on a Site-to-Site VPN on a **virtual private gateway** (must use **transit gateway**)."
- **SAA-C03 relevance:** HIGH (VPN/TGW selection)
- **Further action:** None

### FOUND-002 — Carrier gateway described as Cloud WAN / telecom partner link
- **Category:** ERROR
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Route Tables → Targets; Gateways → Types
- **Original claim:** "Carrier Gateway \- out to managed Wide Are Network (WAN) via AWS Cloud WAN"; "Carrier Gateway \- Connect to AWS partnered telecom network"
- **Issue:** A carrier gateway is an AWS Wavelength feature (carrier network and internet access for Wavelength Zone subnets), not related to Cloud WAN. The second description is roughly right but imprecise.
- **Official evidence:** "A carrier gateway ... allows inbound traffic from a carrier network in a specific location, and ... outbound traffic to the carrier network and the internet... Carrier gateways are only available for VPCs that contain subnets in a Wavelength Zone."
- **Source:** Carrier gateway for AWS Wavelength, https://docs.aws.amazon.com/wavelength/latest/developerguide/carrier-gateways.html
- **Assessment:** The Cloud WAN link is incorrect.
- **Suggested correction:** Route target: "Carrier Gateway \- out to a telecom carrier network (AWS Wavelength Zones)".
- **SAA-C03 relevance:** LOW
- **Further action:** None

### FOUND-003 — Elastic IP / public IPv4 charging is outdated
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Definition & Key Features (cost lists); Default Route; IPs → Elastic IPs
- **Original claim:** "Charged at \$1 for each unassociated address"; "IPv4 is now charged" (default route section); cost lists name "IPv4 addresses, Elastic IPs" without detail
- **Issue:** Since 1 Feb 2024 all public IPv4 addresses (in-use, Elastic, associated or idle) are charged $0.005/hour (~$3.65/month). Idle EIPs were already $0.005/h, not "$1". The "IPv4 is now charged" line in the Default Route section is a mislocated/unclear statement (0.0.0.0/0 itself is not charged). Also, the EIP bullet "Drawn from Amazon's pool (and include the public IPv4 charge)" is fine, but conflicts with the "$1 unassociated" bullet.
- **Official evidence:** "Effective February 1, 2024 there will be a charge of $0.005 per IP per hour for all public IPv4 addresses, whether attached to a service or not"; table includes in-use public IPv4 and EIP assigned to VPC resources, Global Accelerator, and Site-to-Site VPN tunnels; idle EIP $0.005. BYOIP addresses are not charged.
- **Source:** New – AWS Public IPv4 Address Charge + Public IP Insights (AWS News Blog), https://aws.amazon.com/blogs/aws/new-aws-public-ipv4-address-charge-public-ip-insights/
- **Assessment:** Supports replacing "$1" and clarifying that all public IPv4 (not just unassociated EIPs) is charged hourly. Only a blog source was inspected (pricing-page tab did not render), but it is an official AWS publication.
- **Suggested correction:** "Charged at $0.005/hour for every public IPv4 address (in use or idle), to encourage IPv6 use / releasing unused addresses." Remove "IPv4 is now charged, to encourage IPv6 use" from the default-route section or move it to the IPs section.
- **SAA-C03 relevance:** MEDIUM (cost optimisation)
- **Further action:** None

### FOUND-004 — Gateway endpoints are free but listed as costing money
- **Category:** ERROR
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Definition & Key Features → "Do cost money: ... VPC endpoints"
- **Original claim:** "**Do** cost money: NAT gateways, VPC endpoints, ..."
- **Issue:** Interface (and GWLB) endpoints cost money; gateway endpoints (S3, DynamoDB) have no additional charge. The endpoints section itself only labels the interface/GWLB types as costly, so the summary line is imprecise and a common exam point (gateway endpoint vs NAT gateway cost).
- **Official evidence:** "Pricing: There is no additional charge for using gateway endpoints."
- **Source:** Gateway endpoints – AWS PrivateLink Guide, https://docs.aws.amazon.com/vpc/latest/privatelink/gateway-endpoints.html
- **Assessment:** Confirms the gap.
- **Suggested correction:** "VPC endpoints (interface and Gateway Load Balancer; gateway endpoints are free)".
- **SAA-C03 relevance:** HIGH
- **Further action:** None

### FOUND-005 — NAT gateway description omits regional NAT gateways and wrongly limits to IPv4
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** MEDIUM
- **Location:** Core Components (NAT Gateways); Gateways → NAT Gateway; NAT → NAT Gateway ("Required per subnet you want to outwardly connect")
- **Original claim:** "NAT Gateway \- Outbound private traffic for IPv**4**"; "Required **per subnet** you want to outwardly connect"
- **Issue:** (a) AWS now offers regional NAT gateways that automatically span AZs without a hosting public subnet (zonal remains the standard type). (b) "Per subnet" is imprecise — a zonal NAT gateway lives in a public subnet and is normally deployed one per AZ, not per private subnet. (c) The note already mentions DNS64/NAT64, so "IPv4" only is a simplification.
- **Official evidence:** "Use regional NAT gateways ... A regional NAT gateway automatically expands across Availability Zones... Unlike standard NAT gateways (referred to as zonal NAT gateways), which operate in a single Availability Zone..." "You do not need a public subnet to host a regional NAT gateway." Regional NAT gateways "do not offer private connectivity." NAT gateway "is for use with IPv4 or IPv6 traffic (using DNS64 and NAT64)".
- **Source:** Regional NAT gateways for automatic multi-AZ expansion, https://docs.aws.amazon.com/vpc/latest/userguide/nat-gateways-regional.html; NAT gateways, https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html
- **Assessment:** Supported. Exam guide (v1.1) references "single shared NAT gateway compared with NAT gateways for each AZ" – the regional type is not in v1.1 and its exam relevance is uncertain.
- **Suggested correction:** Add one line: "Zonal NAT gateways live in one AZ (use one per AZ for HA); a regional NAT gateway automatically spans AZs." Reword "per subnet" to "per AZ (zonal)".
- **SAA-C03 relevance:** MEDIUM
- **Further action:** Review exam-scope interpretation

### FOUND-006 — Transit gateway bandwidth figure outdated
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** MEDIUM
- **Location:** Transit Gateway
- **Original claim:** "Each attachment handles **50GB/s** of traffic"
- **Issue:** The documented quota is now up to 100 Gbps per VPC attachment per AZ, each direction. Also, the unit "GB/s" should be Gbps (note's units are loose throughout).
- **Official evidence:** "Bandwidth per VPC attachment per Availability Zone: Up to 100 Gbps each direction (i.e., 100 Gbps ingress and 100 Gbps egress)". 5,000 attachments per transit gateway also confirmed.
- **Source:** AWS Transit Gateway Quotas, https://docs.aws.amazon.com/vpc/latest/tgw/transit-gateway-quotas.html
- **Assessment:** The 50 figure is no longer the documented value; exact number unlikely to be examined.
- **Suggested correction:** "Each VPC attachment supports up to 100 Gbps per AZ (each direction)"; consider avoiding the number.
- **SAA-C03 relevance:** LOW
- **Further action:** None

### FOUND-007 — Direct Connect bandwidth options incomplete / unit ambiguity
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** AWS Direct Connect
- **Original claim:** "**Lower** (50MB-500MB/s), **Higher** (1GB, 10GB, 100GB/s)"
- **Issue:** Units should be bits (Mbps/Gbps), not bytes. Options have expanded: hosted connections 50–500 Mbps and 1, 2, 5, 10, 25 Gbps; dedicated connections 1, 10, 100, 400 Gbps. The note's Lower/Higher split also ignores the dedicated vs hosted distinction that it describes later under pricing.
- **Official evidence:** Hosted: "50 Mbps, 100 Mbps, 200 Mbps, 300 Mbps, 400 Mbps, 500 Mbps, 1 Gbps, 2 Gbps, 5 Gbps, 10 Gbps, and 25 Gbps". Dedicated: "1 Gbps, 10 Gbps, 100 Gbps, and 400 Gbps."
- **Source:** Hosted Direct Connect connections, https://docs.aws.amazon.com/directconnect/latest/UserGuide/hosted_connection.html; Dedicated Direct Connect connections, https://docs.aws.amazon.com/directconnect/latest/UserGuide/dedicated_connection.html
- **Assessment:** Supported.
- **Suggested correction:** "Hosted (via partner): 50 Mbps–500 Mbps (and up to 25 Gbps from some partners); Dedicated: 1, 10, 100, 400 Gbps."
- **SAA-C03 relevance:** MEDIUM
- **Further action:** None

### FOUND-008 — NAT instance paragraph contradicts itself
- **Category:** AMBIGUOUS
- **Verification status:** UNVERIFIED
- **Confidence:** MEDIUM
- **Location:** NAT → "Nat Instances"
- **Original claim:** "...legacy deployments of NAT onto individual EC2 instances which have not been deprecated... community AMIs to launch NAT instances to replace the deprecated Amazon one."
- **Issue:** Says both "not deprecated" and "deprecated Amazon one". The current NAT instance docs instruct customers to create their own NAT AMI (the page inspected lists "Create a NAT AMI"), rather than using community AMIs; the prior Amazon-provided NAT AMI is not available. Exact deprecation wording was not located.
- **Official evidence:** NAT instance how-to page inspected shows step "Create a NAT AMI" ("A NAT AMI is configured to run NAT on an EC2 instance. You must create a NAT AMI and then launch..."), plus disabling source/destination check. No statement about deprecation was read in the portion fetched.
- **Source:** Enable private resources to communicate outside the VPC (NAT instances), https://docs.aws.amazon.com/vpc/latest/userguide/work-with-nat-instances.html
- **Assessment:** Evidence supports that you build your own NAT AMI; does not conclusively document the AMI deprecation or the community-AMI statement.
- **Suggested correction:** Verify, then reword to "NAT instances are customer-managed; you create your own NAT AMI (the Amazon-provided NAT AMI is no longer offered) and must disable the source/destination check." Also add that NAT gateways are generally preferred (no scaling/patching) – exam guide mentions NAT instance vs gateway cost.
- **SAA-C03 relevance:** MEDIUM (NAT instance vs gateway comparison is in the exam guide)
- **Further action:** Verify further

### FOUND-009 — Bastion section: AWS-managed alternatives omitted; "AWS does not have its own Bastions" is misleading
- **Category:** INCOMPLETE
- **Verification status:** CONFIRMED
- **Confidence:** MEDIUM
- **Location:** Bastion / Jumpbox
- **Original claim:** "AWS does **not** have its own Bastions... System Manager's **Sessions Manager** can replace the need for a Bastion."
- **Issue:** EC2 Instance Connect Endpoint is an AWS feature that lets you connect to instances without a bastion, public IP or IGW. Systems Manager Session Manager remains valid (note's "System Manager's Sessions Manager" naming is slightly off: Systems Manager / Session Manager).
- **Official evidence:** "EC2 Instance Connect Endpoint allows you to connect securely to an instance from the internet, without using a bastion host, or requiring that your VPC has direct internet connectivity... without requiring the instances to have a public IPv4 or IPv6 address... no additional cost."
- **Source:** Connect to your instances using a private IP address and EC2 Instance Connect Endpoint, https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connect-with-ec2-instance-connect-endpoint.html
- **Assessment:** Supports adding a short mention. The existing statement is not false in that AWS has no dedicated "bastion" product.
- **Suggested correction:** Append: "EC2 Instance Connect Endpoint is another option (no bastion/public IP needed)."
- **SAA-C03 relevance:** MEDIUM
- **Further action:** None

### FOUND-010 — VPC endpoint types incomplete; GWLB endpoint is its own type
- **Category:** INCOMPLETE
- **Verification status:** CONFIRMED
- **Confidence:** MEDIUM
- **Location:** VPC Endpoints → "3 types"; GWLB endpoints "Type of Interface Endpoint"
- **Original claim:** "3 types: Interface, Gateway, Gateway Load Balancer endpoints"; GWLB endpoint is "Type of **Interface Endpoint**"; "Cost money \- per hour & data processed"
- **Issue:** Docs list Interface, GatewayLoadBalancer, Resource, Tunnel, Service network, and Gateway endpoint types. GWLB endpoints are listed as a separate type (the note's "type of interface endpoint" is loose though they use PrivateLink). Also S3 and DynamoDB now support interface endpoints as well as gateway endpoints, so "Gateway... connects to only S3 and DynamoDB" is true but "only gateway for S3/DynamoDB" must not be inferred. The "Bidirectional/Unidirectional" labels were not verified in docs.
- **Official evidence:** "There are multiple types of VPC endpoints: Interface, GatewayLoadBalancer, Resource, Tunnel, Service network ... There is another type of VPC endpoint, Gateway ... Amazon S3 and DynamoDB support both gateway endpoints and interface endpoints."
- **Source:** AWS PrivateLink concepts, https://docs.aws.amazon.com/vpc/latest/privatelink/concepts.html; Gateway endpoints, https://docs.aws.amazon.com/vpc/latest/privatelink/gateway-endpoints.html
- **Assessment:** Core three remain the exam-relevant ones; add one line about S3/DynamoDB interface endpoints (needed for on-premises/peered access) and fix "type of interface endpoint".
- **Suggested correction:** Add: "S3 and DynamoDB also support interface endpoints (PrivateLink)." Reword GWLB endpoint bullet to "Own endpoint type, uses PrivateLink". Resource/service-network endpoints need not be added.
- **SAA-C03 relevance:** HIGH (gateway vs interface endpoint)
- **Further action:** Verify further (bidirectional/unidirectional labels)

### FOUND-011 — "Shared VPC" wording: only subnets shared; wrong CLI/permissions details
- **Category:** AMBIGUOUS
- **Verification status:** UNVERIFIED
- **Confidence:** LOW
- **Location:** Shared VPCs
- **Original claim:** "Sharing does **not** necessarily mean **visibility** of resources which have been shared with you, **cross-account roles** are required."; "You must enable sharing within the organisation via the RAM API."
- **Issue:** Official docs state participants can view/create/modify/delete their own resources in shared subnets and cannot see other participants' resources; the owner can see ENIs and SGs of participants. "Cross-account roles are required" is not supported by the page inspected. Docs also say sharing is with accounts "that belong to the same organization".
- **Official evidence:** "Participants cannot view, modify, or delete resources that belong to other participants or the VPC owner." (overview page only; sub-pages on prerequisites/default subnets not read)
- **Source:** Share your VPC subnets with other accounts, https://docs.aws.amazon.com/vpc/latest/userguide/vpc-sharing.html
- **Assessment:** Overview partially contradicts the "cross-account roles" sentence but the specific prerequisites page was not read.
- **Suggested correction:** Verify on the prerequisites/limitations pages before changing.
- **SAA-C03 relevance:** MEDIUM
- **Further action:** Verify further

### FOUND-012 — Security group reference across peering is region-limited
- **Category:** INCOMPLETE
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** VPC Peering
- **Original claim:** "**Can** reference security groups in peered VPC in your security group rules"
- **Issue:** Not possible for inter-Region peering; cross-account same-Region requires account ID prefix.
- **Official evidence:** "You can't reference the security group of a peer VPC that's in a different Region. Instead, use the CIDR block of the peer VPC."
- **Source:** Update your security groups to reference peer security groups, https://docs.aws.amazon.com/vpc/latest/peering/vpc-peering-security-groups.html
- **Assessment:** The note is true only for same-Region peering.
- **Suggested correction:** "Can reference security groups in a peered VPC (same Region only)."
- **SAA-C03 relevance:** MEDIUM
- **Further action:** None

### FOUND-013 — Peering data transfer cost wording
- **Category:** AMBIGUOUS
- **Verification status:** CONFIRMED
- **Confidence:** MEDIUM
- **Location:** VPC Peering
- **Original claim:** "Data transfer **cross** **AZ** or **region** **costs**"
- **Issue:** Correct but easily misread; the docs state that traffic within the same AZ is free, including cross-account; charges apply across AZs and Regions. Note also earlier says peering is cost-free "itself", consistent.
- **Official evidence:** "There is no charge to create a VPC peering connection. All data transfer over a VPC peering connection that stays within an Availability Zone is free... Charges apply for data transfer over VPC peering connections that cross Availability Zones and Regions."
- **Source:** What is VPC peering?, https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html
- **Assessment:** Note matches evidence; low priority. Reported as AMBIGUOUS only because it omits that same-AZ transfer is free.
- **Suggested correction:** Optional: "Same-AZ data transfer free; cross-AZ and cross-Region charged."
- **SAA-C03 relevance:** LOW
- **Further action:** None

### FOUND-014 — Peered NAU quota applies to same-Region peering only
- **Category:** INCOMPLETE
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Network Address Usage
- **Original claim:** "**Peered VPCs** limited to **128**,000 NAUs (512,000 with increase) \- applies to **total** of peered VPCs"
- **Issue:** The quota applies to same-Region peerings (including cross-account); cross-Region peered VPCs do not contribute. Table also omits Gateway Load Balancer per AZ (6), EFA interface (1) and EKS pod (1) – low importance; "NWLB" should be NLB.
- **Official evidence:** "This quota applies to peerings between VPCs in the same Region, including peerings between VPCs in different AWS accounts. VPCs that are peered across different Regions do not contribute to this quota." Defaults 64,000 / 128,000; increases to 256,000 / 512,000 confirmed.
- **Source:** Network Address Usage for your VPC, https://docs.aws.amazon.com/vpc/latest/userguide/network-address-usage.html
- **Assessment:** Supported.
- **Suggested correction:** Add "(same-Region peering only)".
- **SAA-C03 relevance:** LOW
- **Further action:** None

### FOUND-015 — Flow Logs destination name
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** VPC Flow Logs → "Logs delivered to any of"
- **Original claim:** "**Kinesis** Data Firehouse"
- **Issue:** Service is now named Amazon Data Firehose (formerly Kinesis Data Firehose). (Spelling "Firehouse" is a typo.) The format listing also covers only the default format; custom formats/more fields exist (not an error).
- **Official evidence:** "Flow log data can be published to ... Amazon CloudWatch Logs, Amazon S3, or Amazon Data Firehose."
- **Source:** Logging IP traffic using VPC Flow Logs, https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html
- **Assessment:** Supported.
- **Suggested correction:** "Amazon Data Firehose".
- **SAA-C03 relevance:** LOW
- **Further action:** None

### FOUND-016 — Site-to-Site VPN: tunnel bandwidth and "2 tunnels / AZ" detail not captured; certificate claim unverified
- **Category:** INCOMPLETE
- **Verification status:** UNVERIFIED
- **Confidence:** LOW
- **Location:** AWS VPN → Customer Gateway ("private certificate provisioned by AWS Certificate Manager"); Limitations
- **Original claim:** "Presents: BPG ASN ..., IP address ..., and private certificate provisioned by AWS Certificate Manager (ACM)"
- **Issue:** The certificate is only used for certificate-based authentication (optional, vs pre-shared key) — wording makes it sound mandatory. Not verified. Separately verified: each connection has two tunnels in different AZs; standard 1.25 Gbps per tunnel; Large Bandwidth Tunnels up to 5 Gbps (TGW/Cloud WAN only), which could be added if bandwidth limits matter.
- **Official evidence:** "Each Site-to-Site VPN connection has two tunnels... Each tunnel terminates in a different Availability Zone"; "Standard bandwidth: Up to 1.25 Gbps per tunnel; Large Bandwidth Tunnel: up to 5 Gbps ... only for VPN connections attached to Transit Gateway or Cloud WAN."
- **Source:** Tunnel options for your AWS Site-to-Site VPN connection, https://docs.aws.amazon.com/vpn/latest/s2svpn/VPNTunnels.html
- **Assessment:** Two-tunnel claim in the notes is correct. The certificate wording needs checking against the Customer Gateway documentation.
- **Suggested correction:** Verify, then add "(optional, for certificate authentication)". Optionally add "1.25 Gbps per tunnel standard".
- **SAA-C03 relevance:** MEDIUM
- **Further action:** Verify further
