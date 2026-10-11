# QA: Networking & Content Delivery

Scope: `notes/edited/networking-other.md` only. Compared against `notes/working/networking-other.md` (diff), `review/verified/networking-other.md` and `review/editing/networking-other.md`. Official evidence re-inspected: CloudFront Functions vs Lambda@Edge comparison table (https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/edge-functions-choosing.html). All other claims were checked against the verification report's cited evidence, not re-fetched.

## Summary

- Overall result: **PASS WITH ISSUES**
- Findings reviewed: 17 (16 applied changes + 1 skipped finding)
- PASS: 13 | PASS WITH NOTE: 3 | FAIL: 0 | NEEDS VERIFICATION: 0 (1 unchanged bullet flagged below) | NOT APPLICABLE: 1
- Unauthorised file changes: none. `git status` shows no modifications to `notes/` (original/working unchanged). The diff between working and edited contains only edits traceable to a finding.
- Unauthorised edits within the file: none found. Every diff hunk maps to FOUND-001 to 017 (excluding 015).
- Application report counts match the diff (16 applied).

## Results

| Finding ID | Outcome        | Issue / evidence                                                                                                                                                                                                   | Recommended action                               |
| ---------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------ |
| FOUND-001  | PASS           | HTTP custom domains = Yes; matches verified finding.                                                                                                                                                               | None                                             |
| FOUND-002  | PASS           | HTTP API keys, per-client rate limiting, usage throttling = No; all three rows changed as verified.                                                                                                                | None                                             |
| FOUND-003  | PASS           | REST private integration with ALB = Yes.                                                                                                                                                                           | None                                             |
| FOUND-004  | PASS           | "up to 200 (default quota, can be increased)". Qualifier preserved.                                                                                                                                                | None                                             |
| FOUND-005  | PASS           | Re-checked against the live comparison table: 30s all events, 128 MB / 10,240 MB, 50 MB, millions of req/s, geolocation "No (viewer) / Yes (origin)", JavaScript only. `<check>` removed. Table formatting intact. | None                                             |
| FOUND-006  | PASS           | "$3000 / month, plus data transfer fees; 1-year subscription commitment". Matches verification evidence.                                                                                                           | None                                             |
| FOUND-007  | PASS WITH NOTE | Cross-zone restriction removed as verified. The "Not supported when using ALB as an accelerator endpoint in AWS Global Accelerator" bullet remains and was not confirmed in verification (see Required actions).   | Manual verification of the retained bullet       |
| FOUND-008  | PASS           | "8 types" matches the 8 listed policies; IP-based entry accurate and concise.                                                                                                                                      | None                                             |
| FOUND-009  | PASS           | WAF attach list now consistent with the REST/HTTP "AWS WAF" row (REST yes, HTTP no). "(including classic)" removed from Firewall Manager list.                                                                     | None                                             |
| FOUND-010  | PASS WITH NOTE | Lambda, OpenID Connect and operation-level caching added as verified. "Operation-level caching" is a bare bullet with no description unlike the neighbouring entries. Not an error.                                | Optional: none required                          |
| FOUND-011  | PASS           | Wording matches the suggested correction; consistent with the Endpoints list.                                                                                                                                      | None                                             |
| FOUND-012  | PASS           | OAC bullet placed under Origin; OAI marked legacy; S3 website endpoint exclusion retained (HIGH-relevance qualifier preserved).                                                                                    | None                                             |
| FOUND-013  | PASS           | Rename recorded with former names retained; endpoint descriptions unchanged.                                                                                                                                       | None                                             |
| FOUND-014  | PASS           | "$100 per month per policy per Region", Config/underlying charges and Shield Advanced waiver all present.                                                                                                          | None                                             |
| FOUND-015  | NOT APPLICABLE | Not applied (UNRESOLVED), consistent with apply rules. OWASP list and "OWASP top 10 protection" wording unchanged.                                                                                                 | Author decision (optional wording clarification) |
| FOUND-016  | PASS WITH NOTE | EC1 to EC2 correct. "ISS" dropped; author intent unknown but "Microsoft Smooth Streaming" is the documented feature name.                                                                                          | None                                             |
| FOUND-017  | PASS           | "Only available via Traffic Flow" replaced as verified; following bias bullets remain consistent.                                                                                                                  | None                                             |

## Required actions

1. **Zonal Shift** (`## Route53` > `### Zonal Shift`): the bullet "Not supported when using ALB as an accelerator endpoint in AWS Global Accelerator" was not found in AWS sources during verification and was left unchanged. Manually verify against ARC/Global Accelerator documentation. Remove only if it is contradicted. Do not rely on it for exam questions until verified.
2. **OWASP Top 10** (`## AWS Web Application Firewall (WAF)` and `## CloudFront`, "can be fronted with AWS WAF for OWASP top 10 protection"): FOUND-015 remains a manual-review item. The wording implies full OWASP coverage, which AWS does not claim, and the list is the 2017 edition. Optional minimal edit by the author, as proposed in the verification report.

No other corrections required. Scope reviewed: the single file `notes/edited/networking-other.md` against its verification and application reports; this is not a full-project QA.
