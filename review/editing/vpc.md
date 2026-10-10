# Application Report: VPC

## Summary

- Notes file: `notes/edited/vpc.md`
- Verification report: `review/verified/vpc.md` (proposed wording for unchanged findings taken from triage `review/findings/vpc.md`)
- Findings reviewed: 16
- Changes applied: 15
- Skipped for low confidence: 0
- Skipped for unresolved or conflicting evidence: 0
- Skipped for ambiguity or text mismatch: 0
- Flagged for manual review: 1 (part of FOUND-010)

## Results

| Finding ID | Action        | Reason |
| ---------- | ------------- | ------ |
| FOUND-001  | APPLIED       | Site-to-Site VPN limitations: IPv6 (not IPv4) unsupported on VGW. |
| FOUND-002  | APPLIED       | Route targets and Gateways lists: carrier gateway now Wavelength/telecom carrier network, Cloud WAN link removed. |
| FOUND-003  | APPLIED       | IPs → Elastic IPs: $1 replaced with $0.005/hour for all public IPv4; misplaced "IPv4 is now charged" line removed from Default Route. Rate rests on AWS News Blog (noted in report limitations). |
| FOUND-004  | APPLIED       | Cost list: gateway endpoints are free. |
| FOUND-005  | APPLIED       | NAT Gateway: "per subnet" replaced with zonal per-AZ / regional line. Part (c) not applied (not supported). |
| FOUND-006  | APPLIED       | Transit Gateway: 100Gbps per VPC attachment per AZ. |
| FOUND-007  | APPLIED       | Direct Connect: hosted vs dedicated options, bit units. |
| FOUND-008  | APPLIED       | NAT instance: AWS NAT AMI unsupported, build own AMI, NAT gateways recommended. |
| FOUND-009  | APPLIED       | Bastion: added EC2 Instance Connect Endpoint. |
| FOUND-010  | APPLIED / MANUAL REVIEW | Added S3/DynamoDB interface endpoint note and reworded GWLB bullet. Bidirectional/Unidirectional labels left unchanged (unverified). |
| FOUND-011  | APPLIED       | Shared VPCs: replaced "cross-account roles" bullet with participant visibility rules. |
| FOUND-012  | APPLIED       | Peering: security group reference is same Region only. |
| FOUND-013  | SKIPPED       | REJECTED; note retained. |
| FOUND-014  | APPLIED       | NAU: peered quota is same-Region peering only. |
| FOUND-015  | APPLIED       | Flow Logs: Amazon Data Firehose. |
| FOUND-016  | APPLIED       | Site-to-Site VPN: BGP ASN (dynamic only), optional certificate, 2 tunnels in different AZs, 1.25Gbps each. |

## Applied changes

- Definition & Key Features (cost list) – FOUND-004
- Default Route; IPs → Elastic IPs – FOUND-003
- Route Tables → Targets; Gateways → Types – FOUND-002
- Shared VPCs (last bullet) – FOUND-011
- Peering (security groups) – FOUND-012
- Gateways/VPN limitations – FOUND-001
- Direct Connect – FOUND-007
- VPC Endpoints → Gateway and GWLB bullets – FOUND-010
- Flow Logs destinations – FOUND-015
- VPN → Customer Gateway and tunnels – FOUND-016
- NAT → NAT Gateway, NAT instances – FOUND-005, FOUND-008
- Bastion – FOUND-009
- Transit Gateway – FOUND-006
- NAU – FOUND-014
