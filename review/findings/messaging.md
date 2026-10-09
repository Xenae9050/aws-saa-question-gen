# Triage: Messaging (notes/working/messaging.md)

## Summary

- Section reviewed: `notes/working/messaging.md` (AppFlow, SNS, SQS, Amazon MQ, MSK)
- Official exam guide checked: AWS Certified Solutions Architect – Associate (SAA-C03) Exam Guide, **Version 1.1** (https://d1.awsstatic.com/training-and-certification/docs-sa-assoc/AWS-Certified-Solutions-Architect-Associate_Exam-Guide.pdf; text extracted from the PDF; no publication date found). The certification page (https://aws.amazon.com/certification/certified-solutions-architect-associate/) was opened but only showed language availability.
- Live verification status: Performed. Official AWS docs pages were fetched and read (2026-10-09). 12 findings CONFIRMED; 1 UNVERIFIED (candidate list).
- Findings by category / status:
  - OUTDATED: 4 (all CONFIRMED) — FOUND-001, 002, 004, 005
  - INCOMPLETE: 4 (CONFIRMED) — FOUND-003, 009, 010, 011
  - ERROR: 1 (CONFIRMED) — FOUND-006
  - AMBIGUOUS: 2 (CONFIRMED) — FOUND-007, 008
  - EXAM SCOPE: 1 (UNRESOLVED) — FOUND-012
  - Unchecked candidates: 1 (UNVERIFIED) — FOUND-013
- Important limitations: Only claims that looked wrong/stale were researched. Exam guide has no feature-level detail, so exam-scope conclusions are interpretive. SNS FIFO sub-pages redirected and could not be read. No notes were modified.
- Exam guide coverage: AppFlow, EventBridge, MQ, SNS, SQS, Step Functions (Application Integration) and MSK (Analytics) are all **explicitly listed** in the in-scope services; "Queuing and messaging concepts (for example, publish/subscribe)" is also listed.

## Findings

### FOUND-001 — SNS Message Data Protection no longer available to new customers
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** "Message Data Protection" section
- **Original claim:** "Safeguards data published to SNS topics by using protection policies to audit, mask, redact or block sensitive information…"
- **Issue:** The feature is described as current; AWS has closed it to new customers.
- **Official evidence:** Page states the feature "will no longer be available to new customers effective on April 30, 2026"; existing customers can keep using it; no enhancements. Recommended alternative is Lambda + Amazon Bedrock Guardrails.
- **Source:** Amazon SNS message data protection availability change — https://docs.aws.amazon.com/sns/latest/dg/sns-message-data-protection-availability-change.html
- **Assessment:** Directly contradicts the implied current availability. Feature description itself is still accurate for existing customers.
- **Suggested correction:** Add one line: "No longer available to new customers from 30 April 2026; existing users unaffected."
- **SAA-C03 relevance:** LOW — not in the exam guide at feature level.
- **Further action:** None

### FOUND-002 — SQS maximum message size is now 1 MiB
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** SQS intro, "Message size"
- **Original claim:** "Message size = 1 byte -> 256KB. Amazon SQS Extended Client Library required for larger messages. Max then of 2GB."
- **Issue:** Max SQS message size is 1 MiB; the Extended Client Library is only needed above that.
- **Official evidence:** "The minimum message size is 1 byte (1 character). The maximum is 1,048,576 bytes (1 MiB). To send messages larger than 1 MiB, you can use the Amazon SQS Extended Client Library… The maximum payload size is 2 GB."
- **Source:** Amazon SQS message quotas — https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/quotas-messages.html
- **Assessment:** 256 KB → 1 MiB; the 2 GB extended limit remains correct.
- **Suggested correction:** "1 byte -> 1 MiB (1,048,576 bytes). Extended Client Library required above 1 MiB (max 2GB)."
- **SAA-C03 relevance:** HIGH — common limits/decoupling question detail.
- **Further action:** None

### FOUND-003 — SNS message size configurable up to 1 MiB; 2 GB claim unchecked
- **Category:** INCOMPLETE
- **Verification status:** CONFIRMED (1 MiB behaviour); 2 GB claim see FOUND-013
- **Confidence:** HIGH
- **Location:** SNS "Messages"
- **Original claim:** "When publishing messages >256KB, need to use SNS Extended Client Library (which goes up to 2GB)."
- **Issue:** Topics can now accept up to 1 MiB natively via `MaximumMessageSize` (default 256 KiB). Above 256 KiB only Firehose, SQS and Lambda subscriptions are supported (max 100 subscriptions).
- **Official evidence:** "Amazon SNS supports message payloads up to 1 MiB… when you configure the `MaximumMessageSize` topic attribute. By default, topics accept messages up to 256 KiB."
- **Source:** Publishing large messages with Amazon SNS — https://docs.aws.amazon.com/sns/latest/dg/large-message-payloads.html
- **Assessment:** The extended library is only needed beyond 1 MiB (library behaviour not inspected).
- **Suggested correction:** "Default 256KB; configurable up to 1MiB via MaximumMessageSize (Firehose/SQS/Lambda subscribers only). Larger needs the Extended Client Library."
- **SAA-C03 relevance:** MEDIUM
- **Further action:** Verify further (extended library limit)

### FOUND-004 — MSK supports KRaft
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** "Amazon MSK" bullets; "ZooKeeper connection string URLs"
- **Original claim:** "Uses Zookeeper servers - Does not support KRaft"; "Zookeeper nodes (Znodes) - manage overall structure of the cluster"
- **Issue:** MSK supports both ZooKeeper and KRaft metadata modes since Kafka 3.7.x; ZooKeeper mode is no longer the only option.
- **Official evidence:** "Amazon MSK supports Apache ZooKeeper or KRaft metadata management modes." KRaft clusters use controllers within Kafka; controllers are included at no extra cost; existing clusters can migrate via `UpdateClusterKafkaVersion`. The doc also recommends `BootstrapServerString` over the ZooKeeper connection string.
- **Source:** MSK Metadata management — https://docs.aws.amazon.com/msk/latest/developerguide/metadata-management.html
- **Assessment:** The "does not support KRaft" statement is false. `describe-cluster` returning `ZookeeperConnectString` remains true for ZooKeeper-mode clusters only.
- **Suggested correction:** "Uses ZooKeeper or KRaft (Kafka 3.7.x+) for metadata. KRaft controllers replace ZooKeeper nodes." Qualify ZooKeeper bullets as "ZooKeeper mode".
- **SAA-C03 relevance:** LOW–MEDIUM
- **Further action:** None

### FOUND-005 — SQS FIFO throughput limit incomplete
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** SQS Types → FIFO
- **Original claim:** "Limited messages (300 transactions per second)"; "Can batch in 10 for more throughput"
- **Issue:** 300 TPS applies per partition, per API action, in non-high-throughput mode. Batching gives up to 3,000 messages/s; high throughput mode raises it substantially (region dependent, e.g. up to 70,000 TPS / 700,000 batched messages/s in us-east-1).
- **Official evidence:** "Each partition in a FIFO queue is limited to 300 transactions per second, per API action… This limit applies specifically to non-high throughput mode… If you use batching, non-high throughput FIFO queues support up to 3,000 messages per second."
- **Source:** Amazon SQS message quotas — https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/quotas-messages.html
- **Assessment:** The 300 figure is correct only in the default mode.
- **Suggested correction:** "300 TPS per API action (3,000 msgs/s with batching) by default; high throughput mode available for higher limits."
- **SAA-C03 relevance:** MEDIUM
- **Further action:** None

### FOUND-006 — A2P protocol list contains "SNS"; delivery-policy grouping imprecise
- **Category:** ERROR
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** "Delivery Policy" → "Application to person (A2P) (SMTP, SNS, Mobile Push)"
- **Original claim:** "(SMTP, SNS, Mobile Push)"
- **Issue:** "SNS" should be SMS. The AWS docs group these as "customer managed endpoints" (SMTP, SMS, mobile push; HTTP/S is also customer managed but supports custom policies). The A2A list also omits that Firehose throttling errors use the customer-managed policy.
- **Official evidence:** AWS managed endpoints: Lambda, SQS (3 immediate, 2×1s, 10 exponential 1–20s, 100,000×20s; 100,015 attempts over 23 days). Customer managed: SMTP, SMS, mobile push (0 immediate, 2×10s, 10 exponential 10–600s, 38×600s; 50 attempts over 6 hours). Footnote re Firehose throttling.
- **Source:** Amazon SNS message delivery retries — https://docs.aws.amazon.com/sns/latest/dg/sns-message-delivery-retries.html
- **Assessment:** The retry numbers in the notes match the docs; only the "SNS" label is wrong.
- **Suggested correction:** "(SMTP, SMS, Mobile Push)". Optionally note totals (100,015 attempts/23 days; 50 attempts/6 hours).
- **SAA-C03 relevance:** LOW–MEDIUM
- **Further action:** None

### FOUND-007 — SNS DLQ attaches to subscription, not topic; CLI-only wording
- **Category:** AMBIGUOUS
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** "Dead Letter Queue"
- **Original claim:** "SNS topic configured to send to a DLQ via the CLI."
- **Issue:** DLQ is configured per subscription (redrive policy), not on the topic, and is available in the console as well as CLI/API. Notes omit that the queue must be in the same account and Region, and that an encrypted DLQ needs a customer-managed KMS key.
- **Official evidence:** "A dead-letter queue is attached to an Amazon SNS subscription (rather than a topic)… subscription and SQS queue must be under the same AWS account and Region… FIFO topic subscriptions use FIFO queues, and standard topic subscriptions use standard queues."
- **Source:** Amazon SNS dead-letter queues — https://docs.aws.amazon.com/sns/latest/dg/sns-dead-letter-queues.html
- **Assessment:** Queue-type matching in the notes is correct.
- **Suggested correction:** "DLQ is configured on the subscription (redrive policy); same account and Region."
- **SAA-C03 relevance:** MEDIUM
- **Further action:** None

### FOUND-008 — "unique message group id" for FIFO ordering is misleading
- **Category:** AMBIGUOUS
- **Verification status:** CONFIRMED
- **Confidence:** MEDIUM
- **Location:** SQS Types → FIFO
- **Original claim:** "Order guaranteed (using unique message group id)"
- **Issue:** Ordering is per message group; the group ID is shared by related messages (not unique per message). `MessageGroupId` is required; deduplication ID/content-based deduplication is what prevents duplicates (5-minute interval).
- **Official evidence:** "`MessageGroupId` is required for FIFO queues…"; "messages within a single group are processed sequentially, distributing your workload across multiple message groups allows for better parallelism". Exactly-once page: "If you retry SendMessage within the 5-minute deduplication interval, Amazon SQS doesn't introduce any duplicates."
- **Source:** SQS message quotas — https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/quotas-messages.html ; Exactly-once processing in Amazon SQS — https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/FIFO-queues-exactly-once-processing.html
- **Assessment:** Interpretation: "unique" likely means "a message group ID is required". Related note "Polling required unique id" is unclear and unverified.
- **Suggested correction:** "Order guaranteed within a message group (MessageGroupId required); dedup via deduplication ID (5-min window)."
- **SAA-C03 relevance:** MEDIUM
- **Further action:** Verify further (the "Polling required unique id" line)

### FOUND-009 — Short polling description incomplete
- **Category:** INCOMPLETE
- **Verification status:** CONFIRMED
- **Confidence:** MEDIUM
- **Location:** "Short vs. Long Polling"
- **Original claim:** "Short good if message required right away"
- **Issue:** Short polling samples a subset of servers and may not return all available messages (false empty responses); long polling (WaitTimeSeconds > 0, max 20s) queries all servers. Notes don't give the 20s maximum.
- **Official evidence:** Short polling "queries a subset of servers… a particular ReceiveMessage request might not return all of your messages"; "The maximum long polling wait time is 20 seconds."
- **Source:** Amazon SQS short and long polling — https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-short-and-long-polling.html
- **Assessment:** Existing cost statement is correct; the missing points are commonly tested.
- **Suggested correction:** Add: "Short polling may return empty even when messages exist. Long polling max wait 20s."
- **SAA-C03 relevance:** HIGH
- **Further action:** None

### FOUND-010 — SNS subscription protocol list omits Data Firehose
- **Category:** INCOMPLETE
- **Verification status:** CONFIRMED
- **Confidence:** MEDIUM
- **Location:** SNS "Subscriptions" → Protocols
- **Original claim:** Protocol list (HTTP/S, Email, Email-JSON, SQS, Lambda, SMS, Platform application endpoint)
- **Issue:** Data Firehose is a subscription protocol (and is listed elsewhere in the notes); also HTTP(S)/email/cross-account subscriptions require confirmation.
- **Official evidence:** Console protocol list includes HTTP/HTTPS, Email/Email-JSON, Firehose, Amazon SQS, AWS Lambda, Platform application endpoint, SMS; note that these require confirmation.
- **Source:** Creating a subscription to an Amazon SNS topic — https://docs.aws.amazon.com/sns/latest/dg/sns-create-subscribe-endpoint-to-topic.html
- **Assessment:** Minor internal inconsistency rather than a factual error.
- **Suggested correction:** Add "Amazon Data Firehose" to the list.
- **SAA-C03 relevance:** LOW
- **Further action:** None

### FOUND-011 — MSK public access / networking applies to Provisioned clusters only
- **Category:** INCOMPLETE
- **Verification status:** CONFIRMED
- **Confidence:** MEDIUM
- **Location:** MSK paragraph "Launched within your VPC…"
- **Original claim:** "Public access can be enabled on clusters after their launch."
- **Issue:** Public access applies to MSK Provisioned clusters (Kafka 2.6.0+), with conditions (public subnets, authentication on, in-cluster/TLS encryption, no plaintext); it cannot be enabled at creation. MSK Serverless requires IAM access control and uses PrivateLink for private connectivity.
- **Official evidence:** "…public access to the brokers of MSK Provisioned clusters running Apache Kafka 2.6.0 or later… you can't turn on public access while creating an MSK cluster." / "MSK Serverless requires IAM access control for all clusters."
- **Source:** https://docs.aws.amazon.com/msk/latest/developerguide/public-access.html ; https://docs.aws.amazon.com/msk/latest/developerguide/serverless.html
- **Assessment:** Notes are correct but unqualified.
- **Suggested correction:** "Public access (Provisioned clusters only) can be enabled after launch."
- **SAA-C03 relevance:** LOW–MEDIUM
- **Further action:** None

### FOUND-012 — Depth of Kafka/ZooKeeper/protocol detail vs exam scope
- **Category:** EXAM SCOPE
- **Verification status:** UNRESOLVED
- **Confidence:** LOW
- **Location:** MSK (Apache Kafka, ZooKeeper, bootstrap/ZooKeeper strings), Amazon MQ (AMQP, MQTT, STOMP)
- **Original claim:** Detailed sections on Apache ZooKeeper users, Kafka history (LinkedIn/Scala), STOMP/telnet.
- **Issue:** The exam guide explicitly lists Amazon MSK and Amazon MQ but gives no protocol-level detail. Detail such as ZooKeeper consumers list or Kafka history is likely beyond SAA-level needs (interpretation, not a verified exclusion). The notes lack SAA-style selection guidance (e.g., SQS vs SNS vs MQ vs MSK/Kinesis; EventBridge/Step Functions absent from this section).
- **Official evidence:** SAA-C03 Exam Guide v1.1 lists "Application Integration: Amazon AppFlow, AWS AppSync, Amazon EventBridge, Amazon MQ, Amazon SNS, Amazon SQS, AWS Step Functions" and "Analytics: … Amazon MSK"; also "Queuing and messaging concepts (for example, publish/subscribe)".
- **Source:** SAA-C03 Exam Guide v1.1 — https://d1.awsstatic.com/training-and-certification/docs-sa-assoc/AWS-Certified-Solutions-Architect-Associate_Exam-Guide.pdf
- **Assessment:** Services are in scope; depth is a judgement call.
- **Suggested correction:** None; consider treating the Kafka/ZooKeeper background as optional.
- **SAA-C03 relevance:** MEDIUM
- **Further action:** Review exam-scope interpretation

### FOUND-013 — Candidates not checked against official sources
- **Category:** AMBIGUOUS / INCOMPLETE
- **Verification status:** UNVERIFIED
- **Confidence:** LOW
- **Location:** Various
- **Original claim / candidates:**
  1. SNS "Extended Client Library… up to 2GB" (see FOUND-003).
  2. SNS FIFO "Delivered exactly once (no duplicates)" — SNS FIFO uses a deduplication window; wording may overstate.
  3. "Message Data Protection… Only supported for standard SNS topics."
  4. SQS "Message Timers… Not supported by FIFO queues."
  5. SQS ABAC section (condition keys, alias wording).
  6. SQS "Amazon-managed SSE" — current terminology is SSE-SQS vs SSE-KMS (the SSE page confirms both names).
  7. SQS Standard queue "Can batch similar to SNS"; AppFlow "over 80 cloud services".
- **Official evidence:** Not inspected (SNS FIFO pages redirected). Items 6 confirmed only as terminology at https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-server-side-encryption.html
- **Source:** n/a
- **Assessment:** Not verified; do not treat as errors.
- **Suggested correction:** Verify further.
- **SAA-C03 relevance:** MEDIUM
- **Further action:** Verify further

## Verified as consistent with current docs (no action)
SQS retention (60s–14 days, default 4 days), visibility timeout (30s default, 0–12h), delay queue (0–900s; per-queue setting not retroactive on standard, retroactive on FIFO), temporary queue benefits, SNS default delivery retry numbers, HTTP/S-only custom delivery policy with arithmetic/exponential/geometric/linear backoff, AppFlow feature list, MSK Serverless pay-for-usage.
