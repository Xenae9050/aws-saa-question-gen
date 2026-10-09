# Verification: VPC (notes/working/vpc.md)

Triage report: `review/findings/vpc.md` (16 findings). Verified 9 Oct 2026 against live-inspected official AWS pages and the SAA-C03 Exam Guide v1.1 (re-downloaded and text-extracted; in-scope list includes Client VPN, Direct Connect, PrivateLink, Site-to-Site VPN, Transit Gateway, VPC, Network Firewall and AWS Wavelength; task statements cover NAT instance vs gateway cost, a single shared NAT gateway vs one per AZ, VPN vs Direct Connect, and peering/TGW).

## 1. Summary table

| Finding ID | Verification outcome | Recommended disposition | Priority | Change since triage? | Confidence |
| ---------- | -------------------- | ----------------------- | -------- | -------------------- | ---------- |
| FOUND-001  | CONFIRMED            | CORRECT                 | HIGH     | NO                   | HIGH       |
| FOUND-002  | CONFIRMED            | CORRECT                 | LOW      | NO                   | HIGH       |
| FOUND-003  | CONFIRMED            | CORRECT                 | MEDIUM   | NO                   | HIGH       |
| FOUND-004  | CONFIRMED            | CORRECT                 | HIGH     | NO                   | HIGH       |
| FOUND-005  | PARTIALLY SUPPORTED  | CLARIFY                 | MEDIUM   | YES                  | MEDIUM     |
| FOUND-006  | CONFIRMED            | CORRECT                 | LOW      | NO                   | HIGH       |
| FOUND-007  | CONFIRMED            | CORRECT                 | MEDIUM   | NO                   | HIGH       |
| FOUND-008  | PARTIALLY SUPPORTED  | CLARIFY                 | MEDIUM   | YES                  | HIGH       |
| FOUND-009  | CONFIRMED            | EXPAND                  | MEDIUM   | NO                   | HIGH       |
| FOUND-010  | CONFIRMED            | EXPAND                  | HIGH     | NO                   | MEDIUM     |
| FOUND-011  | PARTIALLY SUPPORTED  | CLARIFY                 | MEDIUM   | YES                  | MEDIUM     |
| FOUND-012  | CONFIRMED            | CORRECT                 | MEDIUM   | NO                   | HIGH       |
| FOUND-013  | REJECTED             | RETAIN                  | LOW      | YES                  | HIGH       |
| FOUND-014  | CONFIRMED            | EXPAND                  | LOW      | NO                   | HIGH       |
| FOUND-015  | CONFIRMED            | CORRECT                 | LOW      | NO                   | HIGH       |
| FOUND-016  | CONFIRMED            | EXPAND                  | MEDIUM   | YES                  | MEDIUM     |

Unchanged rows were re-checked against the same official pages as triage (VPC User Guide route-table options, Wavelength carrier gateways, gateway endpoints, TGW quotas, Direct Connect hosted/dedicated connection pages, EC2 Instance Connect Endpoint, VPC peering security groups, NAU, Flow Logs, AWS News Blog public IPv4 announcement). Only the EIP/public IPv4 rate rests on a blog source (see limitations).

## 2. Details for changed findings

### FOUND-005 — NAT gateway: regional type missing; "per subnet" imprecise

**Change from triage:** Part (c) of the triage ("IPv4 only is a simplification") is not an error. Triage confidence/scope otherwise stands.
**Original assessment:** OUTDATED – omits regional NAT gateways, "per subnet" imprecise, IPv4-only wording too narrow.
**Verified conclusion:** (a) and (b) are supported. (c) is not: the note's "IPv4" labels describe the primary function and the note itself covers DNS64/NAT64 later, so no correction is needed there.
**Official evidence:** Regional NAT gateways automatically expand across AZs, need no public subnet, and "do not support private NAT" (use zonal for private NAT). Zonal NAT gateways "operate in a single Availability Zone" and are created in a public subnet. The comparison page says to "create a NAT gateway in each Availability Zone to ensure zone-independent architecture". The exam guide still frames the skill as single shared NAT gateway vs one per AZ, so the zonal-per-AZ point is the exam-relevant fix; regional is optional context.
**Source:** Regional NAT gateways for automatic multi-AZ expansion, https://docs.aws.amazon.com/vpc/latest/userguide/nat-gateways-regional.html; Compare NAT gateways and NAT instances, https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-comparison.html; NAT gateways, https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html
**Recommended disposition:** CLARIFY
**Proposed replacement:** Replace "Required **per subnet** you want to outwardly connect" with "Zonal NAT gateways sit in one AZ (in a public subnet) – deploy one **per AZ** for availability. A **regional** NAT gateway spans AZs automatically (no public subnet needed, public connectivity only)."
**Remaining uncertainty:** Regional NAT gateway is not in Exam Guide v1.1; exam weight is unclear, so keep it to one line.

### FOUND-008 — NAT instance paragraph

**Change from triage:** Triage UNVERIFIED; now resolved. The "contradiction" is not a real contradiction, and the community-AMI claim is unsupported.
**Original assessment:** AMBIGUOUS – says both "not deprecated" and "deprecated Amazon one"; community AMI statement unverified.
**Verified conclusion:** NAT instances as a pattern are still documented (not deprecated); the AWS-provided NAT AMI is built on Amazon Linux AMI 2018.03, which is out of support. AWS recommends NAT gateways, and tells you to build your own NAT AMI from current Amazon Linux if you need a NAT instance. Docs do not mention community AMIs.
**Official evidence:** "NAT AMI is built on the last version of the Amazon Linux AMI, 2018.03, which reached the end of standard support on December 31, 2020 and end of maintenance support on December 31, 2023 … AWS recommends that you migrate to a NAT gateway … If NAT instances are a better match … you can create your own NAT AMI from a current version of Amazon Linux." The comparison page: "We recommend that you use NAT gateways because they provide better availability and bandwidth and require less effort on your part to administer."
**Source:** NAT instances, https://docs.aws.amazon.com/vpc/latest/userguide/VPC_NAT_Instance.html; Compare NAT gateways and NAT instances, https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-comparison.html
**Recommended disposition:** CLARIFY
**Proposed replacement:** Replace the last sentence with: "The AWS-provided NAT AMI is no longer supported; to use a NAT instance you must create your own NAT AMI (and disable source/destination check). AWS recommends NAT gateways instead."
**Remaining uncertainty:** None material.

### FOUND-011 — Shared VPC visibility / "cross-account roles" wording

**Change from triage:** Triage UNVERIFIED/LOW; prerequisites and permissions pages now read.
**Original assessment:** AMBIGUOUS – "cross-account roles are required" unsupported; RAM-API wording unchecked.
**Verified conclusion:** The "cross-account roles are required" sentence is not supported and is misleading. Sharing gives participants access to the shared subnets; participants see only their own resources (plus owner-created networking objects via describe), not other participants'. "Cannot share default subnets" is correct. "Enable sharing via the RAM API" is not wrong (console or CLI/API, management account) and needs no change.
**Official evidence:** "Participants cannot view, modify, or delete resources that belong to other participants or the VPC owner." Owners "can describe" participants' ENIs and security groups but cannot otherwise work with them. "VPC owners can't share subnets that are in a default VPC." Prerequisites list Organizations-managed accounts, enabling RAM sharing from the management account, and creating a resource share; no cross-account role is listed.
**Source:** Share your VPC subnets with other accounts, https://docs.aws.amazon.com/vpc/latest/userguide/vpc-sharing.html; Shared subnet prerequisites, https://docs.aws.amazon.com/vpc/latest/userguide/vpc-share-prerequisites.html; Responsibilities and permissions, https://docs.aws.amazon.com/vpc/latest/userguide/vpc-share-limitations.html; Sharing your AWS resources (RAM), https://docs.aws.amazon.com/ram/latest/userguide/getting-started-sharing.html
**Recommended disposition:** CLARIFY
**Proposed replacement:** Replace the last bullet with: "Participants can only see and manage their own resources in a shared subnet – not other participants' or the owner's. The owner manages the subnets, route tables, NACLs and gateways."
**Remaining uncertainty:** Docs do not explicitly say roles are "not needed", so the replacement states visibility rules rather than denying roles.

### FOUND-013 — Peering data transfer cost wording

**Change from triage:** Allegation rejected; the note is accurate.
**Original assessment:** AMBIGUOUS – wording easily misread, omits that same-AZ transfer is free.
**Verified conclusion:** "Data transfer cross AZ or region costs" says exactly what the docs say; no free-vs-charged error or misleading implication. The note's earlier "peering (itself)" is free is also consistent.
**Official evidence:** "There is no charge to create a VPC peering connection. All data transfer over a VPC peering connection that stays within an Availability Zone is free, even if it's between different accounts. Charges apply for data transfer over VPC peering connections that cross Availability Zones and Regions."
**Source:** What is VPC peering?, https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html (section "Pricing for a VPC peering connection")
**Recommended disposition:** RETAIN
**Proposed replacement:** None proposed.
**Remaining uncertainty:** None.

### FOUND-016 — Site-to-Site VPN customer gateway / tunnel details

**Change from triage:** Triage UNVERIFIED/LOW; customer gateway options page now inspected and confirms the certificate is optional.
**Original assessment:** INCOMPLETE – certificate wording sounds mandatory; tunnel bandwidth not captured.
**Verified conclusion:** The ACM private certificate is optional (certificate-based authentication only). The BGP ASN is for dynamic routing only. The note's "2 tunnels" is correct, and standard bandwidth is 1.25 Gbps per tunnel (the exam guide's "single VPN compared with multiple VPNs" skill makes this useful).
**Official evidence:** "(Optional) Private certificate from a subordinate CA using AWS Certificate Manager (ACM). If you want to use certificate based authentication…"; "(Dynamic routing only) … BGP ASN"; "Each Site-to-Site VPN connection has two tunnels … each tunnel terminates in a different Availability Zone"; "Standard bandwidth: Up to 1.25 Gbps per tunnel (default)"; Large Bandwidth Tunnels up to 5 Gbps only for TGW/Cloud WAN attachments, not virtual private gateway.
**Source:** Customer gateway options, https://docs.aws.amazon.com/vpn/latest/s2svpn/cgw-options.html; Tunnel options, https://docs.aws.amazon.com/vpn/latest/s2svpn/VPNTunnels.html
**Recommended disposition:** EXPAND
**Proposed replacement:** "Presents: BGP ASN (dynamic routing only), public IP address of the customer gateway device, and optionally a private certificate from AWS Private CA via ACM (certificate-based authentication)." Add: "Each VPN connection has 2 tunnels (different AZs), up to 1.25 Gbps each."
**Remaining uncertainty:** None material. Large Bandwidth Tunnels need not be added.

## 3. Notes on unchanged findings worth knowing

- FOUND-010: Verified that the docs list Interface, GatewayLoadBalancer, Resource, Tunnel, Service network and Gateway as separate endpoint types, and that S3 and DynamoDB support both gateway and interface endpoints. The "Bidirectional"/"Unidirectional" labels in the notes could not be confirmed or refuted from the pages inspected (docs only say the consumer initiates the connection); recommend not changing or relying on those labels without further evidence.
- FOUND-002: Wavelength is in the SAA-C03 in-scope list, but carrier-gateway detail is unlikely to be tested; the incorrect Cloud WAN link should still be fixed.
- FOUND-003: The EC2 Elastic IP page confirms all public IPv4 addresses are charged, in use or idle; the $0.005/hour rate is from the AWS News Blog announcement.

## 4. Totals

- CONFIRMED: 12 (FOUND-001, 002, 003, 004, 006, 007, 009, 010, 012, 014, 015, 016)
- PARTIALLY SUPPORTED: 3 (FOUND-005, 008, 011)
- REJECTED: 1 (FOUND-013)
- UNRESOLVED: 0
- Materially changed findings: 5 (FOUND-005, 008, 011, 013, 016)

### Live-verification limitations

- The VPC pricing page "Public IPv4 Address" tab did not render (the page fetched showed only IPAM pricing), so the current $0.005/hour rate is based on the official AWS News Blog announcement; the EC2 EIP documentation confirms that all public IPv4 addresses are charged but defers to the pricing page for the rate.
- The Exam Guide has no date, only "Version 1.1".
- Gateway/Interface endpoint directionality labels (FOUND-010) remain unverified.
- Not examined (outside triage): VPC Lattice, Traffic Mirroring, Network Firewall, Client VPN content in the notes.
