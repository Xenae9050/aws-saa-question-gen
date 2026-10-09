# Verification: Messaging (notes/working/messaging.md)

Triage report: `review/findings/messaging.md`. Verified 2026-10-09 against live-fetched official AWS pages. Study notes were not modified.

## 1. Summary table

| Finding ID | Verification outcome | Recommended disposition | Priority | Change since triage? |
| ---------- | -------------------- | ----------------------- | -------- | -------------------- |
| FOUND-001  | CONFIRMED            | CORRECT                 | LOW      | NO                   |
| FOUND-002  | CONFIRMED            | CORRECT                 | HIGH     | NO                   |
| FOUND-003  | CONFIRMED            | CORRECT                 | MEDIUM   | YES                  |
| FOUND-004  | CONFIRMED            | CORRECT                 | LOW      | NO                   |
| FOUND-005  | CONFIRMED            | CORRECT                 | MEDIUM   | NO                   |
| FOUND-006  | CONFIRMED            | CORRECT                 | MEDIUM   | NO                   |
| FOUND-007  | CONFIRMED            | CLARIFY                 | MEDIUM   | NO                   |
| FOUND-008  | CONFIRMED            | CLARIFY                 | MEDIUM   | NO                   |
| FOUND-009  | CONFIRMED            | EXPAND                  | HIGH     | NO                   |
| FOUND-010  | CONFIRMED            | EXPAND                  | LOW      | NO                   |
| FOUND-011  | CONFIRMED            | CLARIFY                 | LOW      | NO                   |
| FOUND-012  | UNRESOLVED           | RETAIN                  | LOW      | NO                   |
| FOUND-013  | PARTIALLY SUPPORTED  | CLARIFY                 | MEDIUM   | YES                  |

Classification of the confirmed findings (technically wrong / outdated / incomplete / correct):

- Technically wrong: FOUND-006 ("SNS" should be "SMS"), FOUND-004 ("does not support KRaft").
- Technically correct but outdated: FOUND-001, 002, 005.
- Technically correct but incomplete or imprecise: FOUND-003, 007, 008, 009, 010, 011.
- Technically correct and should remain unchanged: FOUND-012 (no evidence for a change).

Exam-scope source: SAA-C03 Exam Guide v1.1 (https://d1.awsstatic.com/training-and-certification/docs-sa-assoc/AWS-Certified-Solutions-Architect-Associate_Exam-Guide.pdf), text extracted and read. It lists Amazon AppFlow, Amazon MQ, Amazon SNS, Amazon SQS (Application Integration) and Amazon MSK (Analytics) as in-scope. Task Statement 2.1 lists "Queuing and messaging concepts (for example, publish/subscribe)" and "Event-driven architectures". The certification page showed no newer exam version.

## 2. Details for changed findings

### FOUND-003 — SNS message size configurable up to 1 MiB

**Change from triage:** The open item (Extended Client Library limit) is now resolved, and the proposed wording is tightened. The triage text said "Firehose/SQS/Lambda subscribers only". That restriction applies only when `MaximumMessageSize` is above 256 KiB.
**Original assessment:** CONFIRMED, INCOMPLETE. Topics accept up to 1 MiB natively. Further verification was needed for the 2 GB library claim.
**Verified conclusion:** The note's "2GB via Extended Client Library" is still correct, but the library is only needed above 1 MiB. The default remains 256 KiB.
**Official evidence:** `MaximumMessageSize` can be set between 1,024 and 1,048,576 bytes (default 262,144). Above 256 KiB, only Amazon Data Firehose, Amazon SQS and AWS Lambda endpoints are supported, with a maximum of 100 subscriptions. At or below 256 KiB, HTTP/S, SMS, email, email-JSON and mobile push are also supported. For larger payloads, the SNS Extended Client Library (Java/Python) "support[s] messages up to 2 GB" and works with Standard and FIFO topics.
**Source:** Publishing large messages with Amazon SNS — https://docs.aws.amazon.com/sns/latest/dg/large-message-payloads.html (intro, "Supported configuration", "Publishing messages larger than 1 MiB with Amazon S3").
**Recommended disposition:** CORRECT
**Proposed replacement:** "Default max message size is 256KB; can be raised up to 1MiB via the MaximumMessageSize topic attribute (above 256KB only Firehose, SQS and Lambda subscriptions are allowed). Larger than 1MiB needs the SNS Extended Client Library (up to 2GB). Extended client uses S3."
**Remaining uncertainty:** None.

### FOUND-013 — Candidates not checked against official sources (now checked)

**Change from triage:** The triage left all candidates UNVERIFIED. They have now been inspected individually. Most turn out to be correct in the notes (false positives). One needs clarification, and two remain unresolved.
**Original assessment:** UNVERIFIED, LOW confidence. Seven candidate statements were listed for follow-up.
**Verified conclusion:** Per-candidate results:

| #   | Note statement                                                            | Result                                                                                                                                                                                                    | Disposition                  |
| --- | ------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------- |
| 1   | SNS Extended Client Library "up to 2GB"                                   | REJECTED (note is correct; see FOUND-003 evidence)                                                                                                                                                        | RETAIN                       |
| 2   | SNS FIFO "Delivered exactly once (no duplicates)"                         | PARTIALLY SUPPORTED: overstated                                                                                                                                                                           | CLARIFY                      |
| 3   | Message Data Protection "Only supported for standard SNS topics"          | REJECTED (note is correct)                                                                                                                                                                                | RETAIN                       |
| 4   | SQS Message Timers "Not supported by FIFO queues"                         | REJECTED (note is correct)                                                                                                                                                                                | RETAIN                       |
| 5   | SQS ABAC section                                                          | PARTIALLY SUPPORTED: definition, "tags and aliases" and the `aws:ResourceTag/environment: prod` example match the docs. The `aws:RequestTag` and `aws:TagKeys` keys were not seen on the inspected pages. | RETAIN (key list UNRESOLVED) |
| 6   | SQS "Amazon-managed SSE or KMS"                                           | Terminology only. The docs use SSE-SQS (SQS-managed keys) and SSE-KMS.                                                                                                                                    | CLARIFY (LOW)                |
| 7   | AppFlow "over 80 cloud services"; SQS standard "can batch similar to SNS" | "Over 80" could not be confirmed: no inspected official page states a connector count. SQS batching (10 messages per batch) confirmed.                                                                    | "Over 80": UNRESOLVED        |

**Official evidence (candidate 2):**

- SNS FIFO topics provide exactly-once delivery only if: the subscribed SQS FIFO queue exists and permits SNS; the consumer deletes the message before the visibility timeout expires; there is no subscription filter policy ("with message filtering, Amazon SNS FIFO topics support at-most-once delivery"); and there are no network disruptions that prevent acknowledgment.
- Deduplication is based on a deduplication ID (or content-based hash) within a 5-minute interval.

**Sources:**

- SNS FIFO deduplication — https://docs.aws.amazon.com/sns/latest/dg/fifo-message-dedup.html
- Message data protection in Amazon SNS ("Amazon SNS supports message data protection for Amazon SNS standard topics only") — https://docs.aws.amazon.com/sns/latest/dg/message-data-protection.html
- SQS message timers ("FIFO queues don't support timers on individual messages") — https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-message-timers.html
- Encryption at rest in Amazon SQS — https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-server-side-encryption.html
- Attribute-based access control for Amazon SQS — https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-abac.html and https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-abac-tagging-resource-control.html
- Amazon AppFlow overview — https://docs.aws.amazon.com/appflow/latest/userguide/what-is-appflow.html (no connector count stated)

**Recommended disposition:** CLARIFY (candidate 2 and, at LOW priority, candidate 6). No change for the others.
**Proposed replacement:**

- SNS FIFO: "Delivered exactly once within the 5-minute deduplication interval (dedup ID or content-based dedup), provided the consumer deletes before the visibility timeout. With subscription filter policies delivery is at-most-once."
- SSE (optional): "SSE-SQS (SQS-managed keys) or SSE-KMS."
  **Remaining uncertainty:** `aws:RequestTag`/`aws:TagKeys` as ABAC condition keys and the AppFlow "over 80" figure were not confirmed from an inspected official page. No correction is proposed for either.

## 3. Notes on findings with unchanged assessments

- FOUND-008: The "Polling required unique id" line remains unresolved. The `ReceiveMessage` API shows a `ReceiveRequestAttemptId` parameter, but the inspected page text did not explain it (truncated). This is probably what the line refers to, but this is not verified, and no replacement is proposed.
- FOUND-012: The exam guide lists the services but not feature-level depth, so whether Kafka/ZooKeeper/protocol background is excessive cannot be established from official sources. No change is recommended. This is an editorial decision for the author. Messaging remains core SAA content (Task Statement 2.1).

## 4. Counts and limitations

- CONFIRMED: 11 (FOUND-001 to 011)
- PARTIALLY SUPPORTED: 1 (FOUND-013)
- REJECTED: 0 at finding level (3 of the 7 FOUND-013 candidates were false positives)
- UNRESOLVED: 1 (FOUND-012)
- Materially changed vs triage: 2 (FOUND-003, FOUND-013)

Live-verification limitations:

- The exam guide PDF has no publication date. v1.1 is the version served at the official URL. The certification page showed no newer version.
- Some doc pages were truncated by the fetch tool. FOUND-006 and FOUND-009 were verified from the sections read.
- The SNS subscription-protocol list link text for Firehose did not render. It was confirmed via the link target (`sns-firehose-as-subscriber`), the "Firehose endpoints only: Subscription role ARN" step, and the large-message page endpoint list.
- The ABAC condition-key list and the AppFlow "over 80" claim could not be confirmed (see FOUND-013). The SQS `ReceiveRequestAttemptId` behaviour was not inspected (see FOUND-008).
