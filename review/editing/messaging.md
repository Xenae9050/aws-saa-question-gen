# Application Report: Messaging

## Summary

- Notes file: notes/edited/messaging.md
- Verification report: review/verified/messaging.md
- Findings reviewed: 13
- Changes applied: 3 (FOUND-003, FOUND-006, FOUND-013 candidate 2)
- Skipped for low confidence: 0
- Skipped for unresolved or conflicting evidence: 1 (FOUND-012; also FOUND-013 candidates 5 and 7)
- Skipped for ambiguity or text mismatch: 8 (FOUND-001, 002, 004, 005, 007, 008, 009, 010, 011 have no explicit replacement text in the report)
- Flagged for manual review: FOUND-004 (KRaft), FOUND-008 (polling unique id), FOUND-013 candidate 6 (optional SSE wording)

## Results

| Finding ID | Action        | Reason                                                                 |
| ---------- | ------------- | ---------------------------------------------------------------------- |
| FOUND-001  | MANUAL REVIEW | No proposed wording in verification report                             |
| FOUND-002  | MANUAL REVIEW | No proposed wording in verification report                             |
| FOUND-003  | APPLIED       | Explicit replacement supplied                                          |
| FOUND-004  | MANUAL REVIEW | Report says note is wrong but gives no replacement                     |
| FOUND-005  | MANUAL REVIEW | No proposed wording in verification report                             |
| FOUND-006  | APPLIED       | "SNS" corrected to "SMS" in A2P protocol list                          |
| FOUND-007  | MANUAL REVIEW | No proposed wording in verification report                             |
| FOUND-008  | MANUAL REVIEW | Unresolved; no replacement proposed                                    |
| FOUND-009  | MANUAL REVIEW | No proposed wording in verification report                             |
| FOUND-010  | MANUAL REVIEW | No proposed wording in verification report                             |
| FOUND-011  | MANUAL REVIEW | No proposed wording in verification report                             |
| FOUND-012  | SKIPPED       | UNRESOLVED, disposition RETAIN                                         |
| FOUND-013  | APPLIED       | Candidate 2 only (SNS FIFO delivery). Candidate 6 is LOW; others retain |

## Applied changes

- SNS > Messages: replaced ">256KB needs Extended Client Library" with default 256KB, 1MiB via MaximumMessageSize (Firehose/SQS/Lambda only above 256KB), Extended Client Library above 1MiB. (FOUND-003)
- SNS > Topics > FIFO: "Delivered exactly once (no duplicates)" qualified with 5-minute dedup interval, visibility timeout condition and at-most-once with filter policies. (FOUND-013)
- SNS > Delivery Policy: A2P list "SNS" to "SMS". (FOUND-006)
