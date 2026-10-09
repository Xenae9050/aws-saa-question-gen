# Triage: Analytics

## Summary

- Section reviewed: `notes/working/analytics.md` (CloudWatch, EventBridge, Kinesis, CloudTrail, tracing/OTEL/ADOT, Prometheus/AMP, Audit Manager, Inspector, Macie, Security Hub, OpenSearch)
- Official exam guide checked: AWS Certified Solutions Architect – Associate (SAA-C03) Exam Guide, **Version 1.1** (PDF, no date shown): https://d1.awsstatic.com/training-and-certification/docs-sa-assoc/AWS-Certified-Solutions-Architect-Associate_Exam-Guide.pdf. The Task Statement 3.5 knowledge list and the "In-scope AWS services" list were read (text extracted locally from the PDF). The AWS certification page was fetched but returned only a language notice.
- Live verification status: Live web access available. 12 of 14 findings CONFIRMED against inspected AWS documentation pages; 1 UNRESOLVED; 1 UNVERIFIED.
- Findings by category / status:
  - ERROR: 2 (2 CONFIRMED)
  - OUTDATED: 8 (7 CONFIRMED, 1 UNVERIFIED)
  - AMBIGUOUS: 2 (1 CONFIRMED, 1 UNRESOLVED)
  - INCOMPLETE: 2 (2 CONFIRMED)
  - EXAM SCOPE: 0 (see FOUND-014 and the FOUND-008 further action)
  - STYLE: 0 (not reported per repository instructions)
- Important limitations:
  - Only claims that looked wrong, stale or exam-relevant were researched. Claims not listed (e.g. EventBridge event fields and pattern types, EC2 metrics/status checks, OTEL/Prometheus descriptions, Kinesis shard read/write limits, 365-day retention, EFO limit of 20, 90-day CloudTrail Event History, high-resolution metrics, 5 targets per rule, 100 rules per bus, CloudWatch Logs `@` fields) were either spot-checked and found consistent with the docs, or not checked; they are not otherwise endorsed.
  - CloudWatch Logs Insights query limits (FOUND-012) could not be verified because the relevant quota pages redirected or did not show the values.
  - Exam-scope conclusions rest on the SAA-C03 guide v1.1 only; no AWS page announcing a newer guide version was found.
  - Other note sections (e.g. `dbs.md`, `storage.md`) were not scanned, so FOUND-014 may already be covered elsewhere.
  - Passages that appear in the notes as images (`![][image3]`) cannot be assessed.

## Findings

### FOUND-001 — Provisioned Kinesis shard limit
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Amazon Kinesis → Kinesis Data Streams → Capacity modes → Provisioned
- **Original claim:** "Max 200 shards"
- **Issue:** There is no fixed 200-shard maximum; it is an account quota that varies by Region and can be raised.
- **Official evidence:** Quotas page: provisioned mode "There's no upper limit. The default shard quota is 20,000 shards per AWS account" in us-east-1, us-west-2 and eu-west-1; "For all other Regions, the default shard quota is 1,000 or 6,000".
- **Source:** Quotas and limits – Amazon Kinesis Data Streams — https://docs.aws.amazon.com/streams/latest/dev/service-sizes-and-limits.html
- **Assessment:** The page directly contradicts the note.
- **Suggested correction:** "Shard count is limited by an adjustable per-account, per-Region quota (default varies by Region)", or remove the number.
- **SAA-C03 relevance:** LOW — exact quotas are unlikely to be tested.
- **Further action:** None

### FOUND-002 — "Can switch between capacity modes at any time"
- **Category:** AMBIGUOUS
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Kinesis Data Streams → "Can switch between capacity modes at any time"
- **Original claim:** "Can switch between capacity modes at any time"
- **Issue:** Switching is rate-limited.
- **Official evidence:** "you can switch between the on-demand and provisioned capacity modes twice within 24 hours."
- **Source:** Quotas and limits – Amazon Kinesis Data Streams — https://docs.aws.amazon.com/streams/latest/dev/service-sizes-and-limits.html
- **Assessment:** The note is broadly right but omits the limit.
- **Suggested correction:** "Can switch between capacity modes (up to twice per 24 hours per stream)".
- **SAA-C03 relevance:** LOW
- **Further action:** None

### FOUND-003 — On-demand mode "2 consumers by default"
- **Category:** AMBIGUOUS
- **Verification status:** UNRESOLVED
- **Confidence:** MEDIUM
- **Location:** Kinesis Data Streams → Capacity modes → On demand
- **Original claim:** "2 consumers by default (enhanced fan-out (EFO) to add 20 more)"
- **Issue:** The docs state EFO limits as registered consumers per stream (20 for On-demand Standard and Provisioned; 50 for On-demand Advantage). I could not find a source for "2 consumers by default"; it may be a garbled version of the 2 MB/s per-shard shared read throughput. The "On-demand" EFO limit is not different from Provisioned.
- **Official evidence:** "With On-demand Standard and Provisioned modes, you can create up to 20 registered consumers (Enhanced Fan-out Limit) for each data stream."
- **Source:** https://docs.aws.amazon.com/streams/latest/dev/service-sizes-and-limits.html
- **Assessment:** The 20 EFO figure is supported; the "2 consumers" figure is not supported by the page inspected.
- **Suggested correction:** Needs author clarification of intent; likely reword as shared-throughput consumers share 2 MB/s per shard, and EFO allows up to 20 registered consumers (applies to both modes).
- **SAA-C03 relevance:** MEDIUM — EFO vs shared throughput is a common selection concept.
- **Further action:** Verify further

### FOUND-004 — Firehose product name
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Kinesis types → "Amazon (Kinesis) Data Firehose"; consumer list "Kinesis Data Firehose"
- **Original claim:** "Amazon (Kinesis) Data Firehose"
- **Issue:** The service is now **Amazon Data Firehose**. Current docs also list Apache Iceberg tables and OpenSearch Serverless as destinations.
- **Official evidence:** "Amazon Data Firehose is a fully managed service for delivering real-time streaming data to destinations such as Amazon S3, Amazon Redshift, Amazon OpenSearch Service, Amazon OpenSearch Serverless, Splunk, Apache Iceberg Tables, and any custom HTTP endpoint…"
- **Source:** What is Amazon Data Firehose? — https://docs.aws.amazon.com/firehose/latest/dev/what-is-this-service.html
- **Assessment:** Name change confirmed; exam questions may use either name.
- **Suggested correction:** "Amazon Data Firehose (formerly Kinesis Data Firehose)".
- **SAA-C03 relevance:** MEDIUM
- **Further action:** None

### FOUND-005 — Managed Service for Apache Flink supports more than SQL
- **Category:** INCOMPLETE
- **Verification status:** CONFIRMED
- **Confidence:** MEDIUM
- **Location:** Kinesis types → "Managed Service for Apache Fink (formerly: Amazon Kinesis Data Analytics)"
- **Original claim:** "Use custom SQL"
- **Issue:** The service processes streams using Java, Scala, Python or SQL (Flink APIs, plus Studio notebooks). "Custom SQL" alone describes the older Kinesis Data Analytics SQL application model. "Fink" is a typo for "Flink" (style only).
- **Official evidence:** "you can use Java, Scala, Python, or SQL to process and analyze streaming data."
- **Source:** What is Amazon Managed Service for Apache Flink? — https://docs.aws.amazon.com/managed-flink/latest/java/what-is.html
- **Assessment:** The name mapping in the note is correct; the capability description is narrow.
- **Suggested correction:** "Run queries/applications against streaming data using Java, Scala, Python or SQL (Apache Flink)".
- **SAA-C03 relevance:** LOW–MEDIUM
- **Further action:** None

### FOUND-006 — CloudTrail Lake availability and Athena claim
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** MEDIUM
- **Location:** AWS CloudTrail
- **Original claim:** "create a Trail which outputs to S3 and requires Amazon Athena to analyse efficiently (CloudTrail Lake uses Athena under the hood)"
- **Issue:** (a) CloudTrail Lake runs its own SQL queries over event data stores (ORC format); Athena is only an optional route via federation. The "uses Athena under the hood" statement is not supported. (b) CloudTrail Lake closes to new customers from 31 May 2026 (current date is after that). The trail → S3 → Athena approach remains valid.
- **Official evidence:** "AWS CloudTrail Lake will no longer be open to new customers starting May 31, 2026… CloudTrail Lake lets you run SQL-based queries on your events… You can federate an event data store… and run SQL queries on the event data using Amazon Athena."
- **Source:** Working with AWS CloudTrail Lake — https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-lake.html
- **Assessment:** Evidence supports both points. The 90-day Event History statement was checked and is correct (https://docs.aws.amazon.com/awscloudtrail/latest/userguide/view-cloudtrail-events.html).
- **Suggested correction:** Remove the parenthetical, or replace with "CloudTrail Lake queries events directly using SQL (no longer open to new customers)".
- **SAA-C03 relevance:** MEDIUM — CloudTrail → S3/Athena is the pattern most likely to be tested.
- **Further action:** None

### FOUND-007 — Composite alarm actions
- **Category:** ERROR
- **Verification status:** CONFIRMED
- **Confidence:** MEDIUM
- **Location:** CloudWatch Alarms
- **Original claim:** "Only action for composite alarms is to publish to an SNS topic."
- **Issue:** Composite alarms can also invoke Lambda and Systems Manager actions.
- **Official evidence:** "Configure actions" for a composite alarm offers Notification (SNS), "Add Lambda action", and "Add Systems Manager action" (OpsItems / Incident Manager).
- **Source:** Create a composite alarm — https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Create_Composite_Alarm.html
- **Assessment:** The console procedure shows the extra actions. The note may reflect behaviour at launch. EC2 and Auto Scaling actions were not shown for composite alarms; I did not verify they are unsupported.
- **Suggested correction:** "Composite alarms can notify via SNS, invoke Lambda, or create Systems Manager actions (not EC2/Auto Scaling actions)" — the parenthetical needs checking against the PutCompositeAlarm API reference before use.
- **SAA-C03 relevance:** LOW
- **Further action:** Verify further (parenthetical only)

### FOUND-008 — AWS Audit Manager closed to new customers
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** AWS Audit Manager
- **Original claim:** Entire section presents the service as generally available.
- **Issue:** Audit Manager is no longer open to new customers; existing customers can continue using it.
- **Official evidence:** "AWS Audit Manager is no longer open to new customers. Existing customers can continue to use the service as normal."
- **Source:** What is AWS Audit Manager? — https://docs.aws.amazon.com/audit-manager/latest/userguide/what-is.html
- **Assessment:** The content of the note remains accurate; the availability caveat is missing. SAA-C03 guide v1.1 still lists Audit Manager under in-scope services, so this is not evidence it has left the exam.
- **Suggested correction:** Add one line: "No longer open to new customers (existing customers unaffected)."
- **SAA-C03 relevance:** MEDIUM — still in the v1.1 in-scope list.
- **Further action:** Review exam-scope interpretation

### FOUND-009 — Amazon Inspector description is the legacy model
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH (current capabilities); LOW (the "699 checks" and "passive scans" statements, unverified)
- **Location:** AWS Inspector
- **Original claim:** "runs a security benchmark against specific EC2 instances… Network and Host assessments… Steps: 1. Install AWS SSM agent… 3. Run an assessment on the target"
- **Issue:** Current Amazon Inspector is a continuous vulnerability management service that automatically discovers and scans EC2 instances, ECR container images and Lambda functions for software vulnerabilities and unintended network exposure; no manual assessment runs need scheduling. EC2 scanning is agent-based (SSM) or agentless (EBS snapshots). The service name is "Amazon Inspector".
- **Official evidence:** "Amazon Inspector is a vulnerability management service that automatically discovers workloads and continually scans them… With Amazon Inspector, you don't need to manually schedule or configure assessment scans." EC2 page: "agent-based… uses the SSM agent, and agentless scanning collects software inventory using Amazon EBS snapshots."
- **Source:** What is Amazon Inspector? — https://docs.aws.amazon.com/inspector/latest/user/what-is-inspector.html ; Scanning Amazon EC2 instances — https://docs.aws.amazon.com/inspector/latest/user/scanning-ec2.html
- **Assessment:** The note's steps and "benchmark" framing match the retired Inspector Classic, not the current service. I did not locate an AWS page stating Inspector Classic's retirement date, so the retirement claim itself is not made here. CIS benchmark scanning exists in Inspector, but the "699 checks" figure was not verified.
- **Suggested correction:** Reframe in one or two sentences: automated, continuous scanning of EC2, ECR images and Lambda for CVEs and network exposure; SSM agent or agentless; findings with risk score. Keep the hardening/CIS paragraph only if verified.
- **SAA-C03 relevance:** HIGH — Inspector (vulnerabilities) vs GuardDuty/Macie selection is a common exam theme.
- **Further action:** Verify further (CIS "699 checks", "passive scans")

### FOUND-010 — Macie described as CloudTrail anomaly detection
- **Category:** ERROR
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Amazon Macie
- **Original claim:** "continuously monitors S3 data access activity for anomalies… Works by using machine learning to analyse CloudTrail logs." plus the alert-type list and "identify the users most at-risk".
- **Issue:** Macie is a sensitive data discovery service (ML and pattern matching over S3 object content, e.g. PII), plus S3 bucket security/access posture evaluation. It is not primarily an S3 access-activity/CloudTrail anomaly tool. The listed alert categories resemble the original 2017 Macie, not the current service.
- **Official evidence:** "Amazon Macie is a data security service that discovers sensitive data by using machine learning and pattern matching, provides visibility into data security risks, and enables automated protection… evaluates and monitors the buckets for security and access control… sensitive data discovery jobs".
- **Source:** What is Amazon Macie? — https://docs.aws.amazon.com/macie/latest/user/what-is-macie.html
- **Assessment:** The core purpose in the note (S3 data access anomalies via CloudTrail) conflicts with the current description. Macie is explicitly in the in-scope list (Security).
- **Suggested correction:** Replace the first paragraph with: Macie uses ML and pattern matching to discover sensitive data (e.g. PII) in S3, and evaluates S3 bucket security/access; findings go to EventBridge/Security Hub. The alert list needs replacing with current finding types (policy findings and sensitive data findings) after checking the Macie findings documentation.
- **SAA-C03 relevance:** HIGH
- **Further action:** Verify further (finding types list)

### FOUND-011 — Security Hub naming
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** MEDIUM
- **Location:** AWS Security Hub
- **Original claim:** "Cloud Security Posture Management (CSPM)… standards = collections of security controls = AWS Config rules"
- **Issue:** The CSPM capability documented under the name "AWS Security Hub CSPM". The note's description of CSPM is otherwise consistent. "Controls = AWS Config rules" was not verified.
- **Official evidence:** Page title "Introduction to AWS Security Hub CSPM"; Macie docs also refer to "AWS Security Hub CSPM".
- **Source:** Introduction to AWS Security Hub CSPM — https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html
- **Assessment:** Naming change confirmed. How the newer Security Hub relates to Security Hub CSPM was not examined, so no further claims are made.
- **Suggested correction:** Add "(now AWS Security Hub CSPM)" to the heading/first line.
- **SAA-C03 relevance:** MEDIUM — exam guide v1.1 uses "AWS Security Hub".
- **Further action:** Verify further (relationship between Security Hub and Security Hub CSPM)

### FOUND-012 — CloudWatch Logs Insights limits
- **Category:** OUTDATED
- **Verification status:** UNVERIFIED
- **Confidence:** LOW
- **Location:** CloudWatch Logs Insights
- **Original claim:** "Single request can query up to 20 log groups, times out after 15 mins and results are available for 7 days."
- **Issue:** These limits may have changed. The docs also now list additional query languages (OpenSearch PPL and SQL) and "Supports all types of logs" is not verified.
- **Official evidence:** Insights supports three query languages (Logs Insights QL, OpenSearch PPL, OpenSearch SQL): https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CWL_AnalyzeLogData_Languages.html. The limit values themselves could not be retrieved.
- **Source:** (see above)
- **Assessment:** No evidence on the specific numbers. Not presented as a confirmed finding.
- **Suggested correction:** Verify against the CloudWatch Logs quotas page; consider removing the numbers (low exam value).
- **SAA-C03 relevance:** LOW
- **Further action:** Verify further

### FOUND-013 — Elasticsearch Service references
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** MEDIUM
- **Location:** CloudWatch Logs ("Stream to ElasticSearch Service (ES)") and Amazon OpenSearch
- **Original claim:** "Stream to ElasticSearch Service (ES) – to use ELK stack"; "Two engines can be deployed: OpenSearch / ElasticSearch… Free and open (for non-enterprise?)"
- **Issue:** The service is Amazon OpenSearch Service; Elasticsearch is only a legacy engine (OSS versions up to 7.10). The "(for non-enterprise?)" licensing note is an unresolved question in the notes.
- **Official evidence:** OpenSearch Service documents "legacy Elasticsearch" versions and "Elasticsearch OSS (up to 7.10, the final open source version of the software)". It also lists OpenSearch Serverless as a deployment option, which the note already mentions.
- **Source:** What is Amazon OpenSearch Service? — https://docs.aws.amazon.com/opensearch-service/latest/developerguide/what-is.html
- **Assessment:** Supports "legacy", not a full removal; the exact current engine-selection UI was not inspected.
- **Suggested correction:** Change "ElasticSearch Service" to "OpenSearch Service"; label Elasticsearch as legacy (≤7.10) and resolve or delete the "?" licensing note.
- **SAA-C03 relevance:** MEDIUM — only OpenSearch Service is in the exam in-scope list.
- **Further action:** None

### FOUND-014 — Core SAA-C03 analytics services not in this section
- **Category:** INCOMPLETE
- **Verification status:** CONFIRMED
- **Confidence:** LOW (the gap exists only if not covered in other notes)
- **Location:** Whole section (and Kinesis)
- **Original claim:** N/A (omission)
- **Issue:** Task Statement 3.5 knowledge covers data analytics/visualization (Athena, Lake Formation, QuickSight), transformation (AWS Glue), streaming (Kinesis), processing (Amazon EMR), and skills such as building/securing data lakes and converting csv to parquet. The in-scope Analytics list names Athena, EMR, Glue, Kinesis, Lake Formation, MSK, OpenSearch Service, QuickSight, Redshift. This section covers Kinesis and OpenSearch (and mentions Athena, Glue, MSK in passing) but none of the others in depth.
- **Official evidence:** SAA-C03 Exam Guide v1.1, Task Statement 3.5 and "In-scope AWS services and features – Analytics".
- **Source:** AWS Certified Solutions Architect – Associate (SAA-C03) Exam Guide v1.1 — https://d1.awsstatic.com/training-and-certification/docs-sa-assoc/AWS-Certified-Solutions-Architect-Associate_Exam-Guide.pdf
- **Assessment:** Guide content is confirmed; whether other note files cover these services was out of scope.
- **Suggested correction:** None; check other notes (e.g. `dbs.md`, `storage.md`) for Athena/Glue/EMR/Redshift/QuickSight/Lake Formation coverage before adding anything.
- **SAA-C03 relevance:** HIGH
- **Further action:** Review exam-scope interpretation
