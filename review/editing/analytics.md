# Application Report: Analytics

## Summary

- Notes file: `notes/edited/analytics.md`
- Verification report: `review/verified/analytics.md`
- Findings reviewed: 14
- Changes applied: 12
- Skipped for low confidence: 0
- Skipped for unresolved or conflicting evidence: 1 (FOUND-003)
- Skipped for ambiguity or text mismatch: 0
- Flagged for manual review: 1 (FOUND-014)

## Results

| Finding ID | Action | Reason |
| --- | --- | --- |
| FOUND-001 | APPLIED | Kinesis provisioned "Max 200 shards" replaced with adjustable-quota wording. |
| FOUND-002 | APPLIED | Capacity mode switching qualified (twice per 24 hours per stream). |
| FOUND-003 | SKIPPED | UNRESOLVED / INVESTIGATE_FURTHER; no replacement proposed. |
| FOUND-004 | APPLIED | Firehose renamed to Amazon Data Firehose (Kinesis types list and consumer list). |
| FOUND-005 | APPLIED | Managed Flink bullet now lists Java, Scala, Python or SQL. |
| FOUND-006 | APPLIED | CloudTrail: corrected Lake/Athena parenthetical and added availability caveat. |
| FOUND-007 | APPLIED | CloudWatch Alarms: composite alarm actions corrected. |
| FOUND-008 | APPLIED | Audit Manager: added availability note under heading. |
| FOUND-009 | APPLIED | AWS Inspector: replaced description paragraph and Steps list; removed unverified "699 checks" and "passive scans". Hardening paragraph kept. |
| FOUND-010 | APPLIED | Amazon Macie: replaced description, Alerts list and final sentence. |
| FOUND-011 | APPLIED | Security Hub: named as Security Hub CSPM; controls "most evaluated using AWS Config rules". |
| FOUND-012 | APPLIED | CloudWatch Logs Insights: 20→50 log groups, 15→60 mins. "Supports all types of logs" left unchanged (unverified). |
| FOUND-013 | APPLIED | CloudWatch Logs stream target renamed to OpenSearch Service; Elasticsearch labelled legacy (OSS up to 7.10). |
| FOUND-014 | MANUAL REVIEW | INVESTIGATE_FURTHER; coverage gap (EMR, QuickSight, MSK) is a scope decision, not a correction. |
