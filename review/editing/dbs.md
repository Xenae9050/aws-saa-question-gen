# Application Report: Databases

## Summary

- Notes file: `notes/edited/dbs.md`
- Verification report: `review/verified/dbs.md`
- Findings reviewed: 14
- Changes applied: 11 (001, 002, 003, 005, 006, 007, 008, 010, 012, 013 partial; 003 touched two places)
- Skipped for low confidence: 2 (009, 014)
- Skipped for unresolved or conflicting evidence: 0
- Skipped for ambiguity or text mismatch: 0
- Flagged for manual review: 2 (011, 015)

## Results

| Finding ID | Action        | Reason |
| ---------- | ------------- | ------ |
| FOUND-001  | APPLIED       | RDS → Multi AZ: separated instance vs DB cluster deployments, Aurora noted as different; comparison table header qualified as "DB Instance". |
| FOUND-002  | APPLIED       | RDS → Encryption: TLS supported on all engines, client-initiated, enforcement engine-specific (`rds.force_ssl`). |
| FOUND-003  | APPLIED       | Aurora Serverless v2 bullet and comparison table: scale to 0 ACUs (newer versions), 0–256 ACUs version dependent. |
| FOUND-005  | APPLIED       | Redshift: Multi-AZ available for RA3/RG. |
| FOUND-006  | APPLIED       | Redshift node types replaced with RA3/RG, DC2, DS2 unavailable; node-count bullet no longer implies one node size. Serverless line not added (unverified). |
| FOUND-007  | APPLIED       | RDS read replicas: 5 → 15 per source. |
| FOUND-008  | APPLIED       | Aurora Global Database: 5 → 10 secondary clusters. |
| FOUND-009  | SKIPPED       | PARTIALLY SUPPORTED, LOW confidence. |
| FOUND-010  | APPLIED       | Aurora: "Serverless Provisioned" → "Provisioned is the traditional/default". |
| FOUND-011  | APPLIED (by user request) | QLDB section reduced to a one-line shutdown note. |
| FOUND-012  | APPLIED       | ElastiCache: added Valkey/Redis OSS, public endpoint caveat; MemoryDB latency wording. |
| FOUND-013  | APPLIED (partial) | DynamoDB Features: added capacity modes. Partition split and DAX/Global Tables not changed (unverified). |
| FOUND-014  | SKIPPED       | PARTIALLY SUPPORTED, LOW confidence. |
| FOUND-015  | MANUAL REVIEW | REMOVE_OR_DEPRIORITISE: Glue Studio CodeCommit bullet and QLDB section; scope depth is interpretive. |
