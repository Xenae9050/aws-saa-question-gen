# Application Report: Storage

## Summary

- Notes file: notes/edited/storage.md
- Verification report: review/verified/storage.md
- Findings reviewed: 14
- Changes applied: 6 (FOUND-002, 006, 010, 011, 013, 014)
- Skipped for low confidence: FOUND-003, 004, 009
- Skipped for unresolved or conflicting evidence: FOUND-009
- Skipped for ambiguity or text mismatch: FOUND-001, 005, 007 (no explicit proposed correction in the verification report; 007 is STYLE)
- Flagged for manual review: FOUND-001, 005, FOUND-008 (80/39.5 TB figures; Snowcone/Snowmobile unverified), Io1 durability "99.8%-99.8%" (unnumbered observation, should be 99.8%-99.9%)

## Results

| Finding ID | Action        | Reason |
| ---------- | ------------- | ------ |
| FOUND-001  | MANUAL REVIEW | HIGH, but proposed correction not stated in report |
| FOUND-002  | APPLIED       | io2 merged with Block Express; use-case cell updated |
| FOUND-003  | SKIPPED       | LOW confidence |
| FOUND-004  | SKIPPED       | LOW; disposition RETAIN |
| FOUND-005  | MANUAL REVIEW | HIGH, but proposed correction not stated in report |
| FOUND-006  | APPLIED       | EFS pricing/storage classes (EXPAND) |
| FOUND-007  | SKIPPED       | STYLE typo only |
| FOUND-008  | MANUAL REVIEW | Partially supported; separating supported portion needs interpretation |
| FOUND-009  | SKIPPED       | UNRESOLVED / LOW |
| FOUND-010  | APPLIED       | Transfer Family ports corrected |
| FOUND-011  | APPLIED       | AMS -> MGN; Migration Hub closure line added |
| FOUND-012  | SKIPPED       | Confirmed; note already correct |
| FOUND-013  | APPLIED       | Cached volume size 1GB-32GB -> 1GiB-32TiB |
| FOUND-014  | APPLIED       | Tape Gateway archive target line added |

## Applied changes

- EBS types list and Provisioned IOPS use-case cell: FOUND-002
- EFS: FOUND-006
- Transfer Families, Common ports: FOUND-010
- Migration Hub: FOUND-011
- Storage Gateway, Cached volumes: FOUND-013
- Storage Gateway, Tape Gateway: FOUND-014
