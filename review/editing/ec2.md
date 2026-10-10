# Application Report: EC2

## Summary

- Notes file: `notes/edited/ec2.md`
- Verification report: `review/verified/ec2.md` (confidence taken from `review/findings/ec2.md`)
- Findings reviewed: 16
- Changes applied: 13 (FOUND-001 to 013)
- Skipped for low confidence: 3 (FOUND-014, 015, 016)
- Skipped for unresolved or conflicting evidence: 0
- Skipped for ambiguity or text mismatch: 0
- Flagged for manual review: 3 (same as low-confidence items)

## Results

| Finding ID | Action | Reason |
| --- | --- | --- |
| FOUND-001 | APPLIED | Hostnames → Resource Name → Other regions: placeholder changed to `[ec2-instance-id]` |
| FOUND-002 | APPLIED | Cloud-Init → Metadata: IPv6 `::245` → `::254` |
| FOUND-003 | APPLIED | Instance Profile: removed reboot claims; no reboot needed; prefer replacing profile (removal up to 1h delay) |
| FOUND-004 | APPLIED | Placement Groups: Spread "7 running instances per AZ"; Partition "max 7 partitions per AZ, multi-AZ" |
| FOUND-005 | APPLIED | Amazon Linux → Versions: AL1 EOL 2023-12-31, AL2 EOS 2026-06-30; TODO removed |
| FOUND-006 | APPLIED | AL2023: SELinux permissive; sourced from multiple Fedora versions and others. Cronie line already correct |
| FOUND-007 | APPLIED | Pricing → Reserved: 75%→72%, Convertible 54%→66% |
| FOUND-008 | APPLIED | Standard RI: deleted EC2-Classic bullet; qualified instance size change |
| FOUND-009 | APPLIED | RI Marketplace: $50,000 lifetime limit. GovCloud bullet untouched (unverified) |
| FOUND-010 | APPLIED | Instance Lifecycle: added Hibernate action. States list not changed (verification gave no concrete state) |
| FOUND-011 | APPLIED | Fargate: awsvpc network mode; awslogs noted as log driver |
| FOUND-012 | APPLIED | Service Connect: successor to App Mesh; App Mesh end-of-support noted; no out-of-scope claim |
| FOUND-013 | APPLIED | Compute Optimizer: 14-day default / 93 days enhanced; added RDS/Aurora and idle resources |
| FOUND-014 | MANUAL REVIEW | LOW confidence; no official ENA bandwidth figure |
| FOUND-015 | MANUAL REVIEW | LOW confidence; partially supported, Spot for Hosts unverified |
| FOUND-016 | MANUAL REVIEW | LOW confidence in triage; author intent unknown |
