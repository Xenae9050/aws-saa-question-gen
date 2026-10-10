# Application Report: Networking & Content Delivery

## Summary

- Notes file: notes/edited/networking-other.md
- Verification report: review/verified/networking-other.md
- Findings reviewed: 17
- Changes applied: 16
- Skipped for low confidence: 0
- Skipped for unresolved or conflicting evidence: 1 (FOUND-015)
- Skipped for ambiguity or text mismatch: 0
- Flagged for manual review: 2 (FOUND-015; Zonal Shift Global Accelerator bullet, unverified, left unchanged)

## Results

| Finding ID | Action  | Reason                                                                                                         |
| ---------- | ------- | -------------------------------------------------------------------------------------------------------------- |
| FOUND-001  | APPLIED | API Management table: HTTP custom domains = Yes                                                                |
| FOUND-002  | APPLIED | API Management table: HTTP API keys, per-client rate limiting and usage throttling = No                        |
| FOUND-003  | APPLIED | Integrations table: REST private integration with ALB = Yes                                                    |
| FOUND-004  | APPLIED | Health Checks: up to 200 (default quota, increasable)                                                          |
| FOUND-005  | APPLIED | Lambda@Edge vs CloudFront Functions table: scale, duration, memory, code size, geolocation row; removed `<check>` |
| FOUND-006  | APPLIED | Shield Advanced: $3000 / month plus data transfer fees; 1-year commitment (confirmed in verification report)    |
| FOUND-007  | APPLIED | Zonal Shift: removed cross-zone-off restriction. GA bullet left unchanged (not verified)                       |
| FOUND-008  | APPLIED | Routing Policies: "8 types"; added IP-based routing                                                            |
| FOUND-009  | APPLIED | WAF attach list: added API Gateway (REST), AppSync, Cognito user pools; Firewall Manager: dropped "(including classic)" |
| FOUND-010  | APPLIED | AppSync: added Lambda and OpenID Connect auth, operation-level caching                                         |
| FOUND-011  | APPLIED | Global Accelerator Custom Routing wording clarified                                                            |
| FOUND-012  | APPLIED | CloudFront Origin: added OAC bullet (OAI legacy; not for S3 website endpoints)                                 |
| FOUND-013  | APPLIED | Resolver renamed "Route 53 VPC Resolver (previously Route 53 Resolver ...)"                                    |
| FOUND-014  | APPLIED | Firewall Manager cost: per policy per Region, plus Config/underlying charges, waived for Shield Advanced       |
| FOUND-015  | MANUAL REVIEW | UNRESOLVED outcome; AWS does not pin an OWASP edition. Optional wording clarification left to author     |
| FOUND-016  | APPLIED | "Amazon EC1" to "Amazon EC2"; "ISS Microsoft Smooth Streaming" to "Microsoft Smooth Streaming" (author intent for "ISS" unknown) |
| FOUND-017  | APPLIED | Geo-proximity: no longer "only via Traffic Flow"; Traffic Flow provides the visual map                         |
