# Application Report: General

## Summary

- Notes file: notes/edited/general.md
- Verification report: review/verified/general.md
- Findings reviewed: 22
- Changes applied: 9 findings (002, 004, 005, 008, 009 partial, 010, 012, 017, 022 partial)
- Skipped for low confidence: 0
- Skipped for unresolved or conflicting evidence: 022 container-image "slower" and 010 "orange box" (no change proposed)
- Skipped for ambiguity or text mismatch: 001, 003, 006, 007, 014, 016, 018, 019, 020, 021 (summary-only in report, no explicit proposed correction)
- Flagged for manual review: 009 (remove "Create organization", "Change or cancel AWS Support plan"), 011, 013, 015 (REMOVE_OR_DEPRIORITISE)

## Results

| Finding ID | Action | Reason |
| ---------- | ------ | ------ |
| FOUND-002 | APPLIED | CLI line: v2 bundles Python; v1 maintenance mode |
| FOUND-004 | APPLIED | STS: https, global legacy default, Regional recommended |
| FOUND-005 | APPLIED | Temporary credentials: up to several hours (roles 12h) |
| FOUND-008 | APPLIED | Renamed heading to AWS Health Dashboard; Service health page, no sign-in |
| FOUND-009 | APPLIED (partial) | Narrowed "Change account settings"; annotated "Close AWS account". Removals rest on absence from list, left for manual review; MFA wording unchanged (optional) |
| FOUND-010 | APPLIED | Principle→Principal (policy elements, Detective "principals"); orange box retained |
| FOUND-011 | MANUAL REVIEW | REMOVE_OR_DEPRIORITISE |
| FOUND-012 | APPLIED | Added Elastic Transcoder discontinued 13 Nov 2025 |
| FOUND-013 | MANUAL REVIEW | REMOVE_OR_DEPRIORITISE |
| FOUND-015 | MANUAL REVIEW | REMOVE_OR_DEPRIORITISE |
| FOUND-017 | APPLIED | Removed Cognito bullet from Directory Service "Offers" |
| FOUND-022 | APPLIED (partial) | Lambda default concurrency qualifier (fixes "functionals"); arm64 wording. Container-image claim untouched |
| Others | SKIPPED | No explicit proposed correction in report |
