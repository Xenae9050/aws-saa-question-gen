# Verification Report: Analytics

## 1. Verification Summary

- **Notes section:** `notes/working/analytics.md`
- **Triage report:** `review/findings/analytics.md`
- **Verification date:** 2026-10-09
- **Official SAA-C03 exam guide:** AWS Certified Solutions Architect – Associate (SAA-C03) Exam Guide, Version 1.1 (no publication date shown in the PDF), https://d1.awsstatic.com/training-and-certification/docs-sa-assoc/AWS-Certified-Solutions-Architect-Associate_Exam-Guide.pdf
- **Findings reviewed:** 14
- **Confirmed findings:** 10 (FOUND-001, 002, 004, 005, 006, 007, 008, 010, 011, 012)
- **Partially supported findings:** 3 (FOUND-009, 013, 014)
- **Rejected findings:** 0
- **Unresolved findings:** 1 (FOUND-003)
- **Live verification limitations:**
  - All AWS documentation pages cited below were fetched live on 2026-10-09. AWS docs pages generally show no publication/update date, so none is recorded.
  - The exam guide PDF was downloaded and its text extracted locally. It states "Version 1.1 SAA-C03" with no date. The AWS certification landing page was not usable (per triage), so the existence of a newer guide version could not be ruled out.
  - The Amazon Inspector Classic user guide (`/inspector/v1/userguide/...`) returned 404, so Inspector Classic's retirement could not be evidenced. Nothing below depends on it.
  - The relationship between the newer "AWS Security Hub" and "AWS Security Hub CSPM" could not be established; the Security Hub user guide landing page returned no usable content.
  - Passages in the notes that are images were not assessed.
  - Claims in the notes that were not part of a triage finding were not verified and are not endorsed.

## 2. Finding-by-finding verification

### FOUND-001 — Provisioned Kinesis shard limit
**Original category:** OUTDATED
**Original triage confidence:** HIGH
**Verification outcome:** CONFIRMED
**Verified confidence:** HIGH

**Original note**

> Provisioned … - Max 200 shards

**Triage allegation**

There is no fixed 200-shard maximum. Shard count is an adjustable per-account, per-Region quota.

**Official sources inspected**

- Title: Quotas and limits – Amazon Kinesis Data Streams
- URL: https://docs.aws.amazon.com/streams/latest/dev/service-sizes-and-limits.html
- Source type: AWS service documentation
- Relevant passage: "Number of shards" row, provisioned mode: "There's no upper limit. The default shard quota is 20,000 shards per AWS account" for US East (N. Virginia), US West (Oregon), Europe (Ireland); "For all other Regions, the default shard quota is 1,000 or 6,000 shards per AWS account." Increases are requested via Service Quotas.
- Publication or update date: Not shown
- Access status: page successfully inspected

**Evidence analysis**

The page explicitly contradicts a fixed 200-shard maximum. The quota is Region-dependent and can be raised. The source is current and specific to the claim.

**Final technical assessment**

Technically correct only for a very old default; now outdated (no fixed maximum).

**Final exam-scope assessment**

Reasonably covered by a broader exam objective. Task Statement 3.5 (streaming data services, e.g. Kinesis; designing data streaming architectures). Exact quotas are unlikely to be tested; the provisioned-vs-on-demand trade-off is.

**Recommended disposition**

CORRECT

**Proposed replacement text**

> Shard count is limited by an adjustable per-account, per-Region quota (no fixed maximum)

**Rationale**

Removes a wrong hard number while keeping the point that the customer manages shards. Quoting the default values is not needed at SAA level.

**Correction priority:** LOW

**Next action:** Ready for correction

---

### FOUND-002 — "Can switch between capacity modes at any time"
**Original category:** AMBIGUOUS
**Original triage confidence:** HIGH
**Verification outcome:** CONFIRMED
**Verified confidence:** HIGH

**Original note**

> Can switch between capacity modes at any time

**Triage allegation**

Switching is rate-limited, so "at any time" is imprecise.

**Official sources inspected**

- Title: Quotas and limits – Amazon Kinesis Data Streams
- URL: https://docs.aws.amazon.com/streams/latest/dev/service-sizes-and-limits.html
- Source type: AWS service documentation
- Relevant passage: "Switching between provisioned and on-demand modes": "For each data stream in your AWS account, you can switch between the on-demand and provisioned capacity modes twice within 24 hours." The same limit appears on the `UpdateStreamMode` row.
- Publication or update date: Not shown
- Access status: page successfully inspected

**Evidence analysis**

The explicit limit (twice per 24 hours per stream) qualifies "at any time". The note's core claim (switching is allowed, no recreation needed) is not contradicted.

**Final technical assessment**

Technically correct but ambiguous/incomplete.

**Final exam-scope assessment**

Reasonably covered by a broader exam objective (Task Statement 3.5). Low exam value.

**Recommended disposition**

CLARIFY

**Proposed replacement text**

> Can switch between capacity modes (up to twice per 24 hours per stream)

**Rationale**

Smallest change that makes the statement precise.

**Correction priority:** LOW

**Next action:** Ready for correction

---

### FOUND-003 — On-demand mode "2 consumers by default"
**Original category:** AMBIGUOUS
**Original triage confidence:** MEDIUM
**Verification outcome:** UNRESOLVED
**Verified confidence:** MEDIUM

**Original note**

> On demand … - 2 consumers by default (enhanced fan-out (EFO) to add 20 more)

**Triage allegation**

No official source supports "2 consumers by default". The EFO limit applies equally to provisioned mode and is a registered-consumer limit, not "20 more".

**Official sources inspected**

- Title: Quotas and limits – Amazon Kinesis Data Streams
- URL: https://docs.aws.amazon.com/streams/latest/dev/service-sizes-and-limits.html
- Source type: AWS service documentation
- Relevant passage: "Number of registered consumers per data stream": On-demand Advantage up to 50 registered consumers (Enhanced Fan-out); "With Kinesis On-Demand Standard and Kinesis Provisioned modes, you can create up to 20 registered consumers (Enhanced Fan-out Limit) for each data stream."
- Publication or update date: Not shown
- Access status: page successfully inspected

- Title: Develop enhanced fan-out consumers with dedicated throughput
- URL: https://docs.aws.amazon.com/streams/latest/dev/enhanced-consumers.html
- Source type: AWS service documentation
- Relevant passage: Shared-throughput consumers: "Fixed at a total of 2 MB/sec per shard. If there are multiple consumers reading from the same shard, they all share this throughput." EFO consumers each get up to 2 MB/sec per shard. Registration limits are 20 (On-demand Standard and Provisioned) or 50 (On-demand Advantage).
- Publication or update date: Not shown
- Access status: page successfully inspected

**Evidence analysis**

The pages establish the 20-consumer EFO limit and that it is not specific to on-demand mode. Neither page states a default of "2 consumers". The figure may be a garbled form of the 2 MB/s shared read throughput per shard, but that is interpretation, not evidence. The intended meaning needs author clarification; absence of the phrase in two pages is not proof the figure is wrong.

**Final technical assessment**

Cannot be reliably assessed as written. The "20" EFO figure is supported; the "2 consumers by default" and the placement under on-demand only are not supported by the inspected sources.

**Final exam-scope assessment**

Reasonably covered by a broader exam objective (Task Statement 3.5, streaming data services). Shared throughput vs enhanced fan-out is a plausible service-selection concept.

**Recommended disposition**

INVESTIGATE_FURTHER

**Proposed replacement text**

No replacement proposed pending further verification.

**Rationale**

The author's intent is unclear and a correction would be speculative. The separate "Enhanced Fan Out (EFO)" section already states the 20-consumer limit correctly for standard modes.

**Correction priority:** LOW

**Next action:** Further research required

---

### FOUND-004 — Firehose product name
**Original category:** OUTDATED
**Original triage confidence:** HIGH
**Verification outcome:** CONFIRMED
**Verified confidence:** HIGH

**Original note**

> - Amazon (Kinesis) Data Firehose

and, in the consumer list:

> Kinesis Data Firehose \- in turn integrates delivery to other AWS services

**Triage allegation**

The service is now Amazon Data Firehose; docs also list Apache Iceberg tables and OpenSearch Serverless as destinations.

**Official sources inspected**

- Title: What is Amazon Data Firehose?
- URL: https://docs.aws.amazon.com/firehose/latest/dev/what-is-this-service.html
- Source type: AWS service documentation
- Relevant passage: "Amazon Data Firehose is a fully managed service for delivering real-time streaming data to destinations such as Amazon S3, Amazon Redshift, Amazon OpenSearch Service, Amazon OpenSearch Serverless, Splunk, Apache Iceberg Tables, and any custom HTTP endpoint…"
- Publication or update date: Not shown
- Access status: page successfully inspected

- Title: Real-time processing of log data with subscriptions (CloudWatch Logs)
- URL: https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/Subscriptions.html
- Source type: AWS service documentation
- Relevant passage: subscription destinations include "an Amazon Kinesis stream, an Amazon Data Firehose stream, or AWS Lambda".
- Publication or update date: Not shown
- Access status: page successfully inspected

**Evidence analysis**

Two separate AWS docs use "Amazon Data Firehose" as the current name. The inspected Firehose page does not itself say "formerly Kinesis Data Firehose"; the rename link is an interpretation from the note's own old name and the new documentation name. The note's functional description is unaffected. The exam guide lists "Amazon Kinesis" generally, so either name may appear in study material.

**Final technical assessment**

Technically correct but outdated (name).

**Final exam-scope assessment**

Reasonably covered by a broader exam objective (Task Statement 3.5, streaming/ingestion; "Amazon Kinesis" in the in-scope Analytics list).

**Recommended disposition**

CORRECT

**Proposed replacement text**

> - Amazon Data Firehose (previously Kinesis Data Firehose)

and "Amazon Data Firehose \- in turn integrates delivery to other AWS services" in the consumer list.

**Rationale**

Aligns the name with current docs while keeping the old name visible for older exam material. The note's own later wording ("Very similar to Data Firehose") already uses the new name.

**Correction priority:** MEDIUM

**Next action:** Ready for correction

---

### FOUND-005 — Managed Service for Apache Flink supports more than SQL
**Original category:** INCOMPLETE
**Original triage confidence:** MEDIUM
**Verification outcome:** CONFIRMED
**Verified confidence:** HIGH

**Original note**

> - Managed Service for Apache Fink (formerly: Amazon Kinesis Data Analytics)
>   - Run queries against data flowing through real-time stream
>   - Create reports and analysis on emerging data
>   - Use custom SQL

**Triage allegation**

"Use custom SQL" is narrow; the service supports Java, Scala, Python and SQL.

**Official sources inspected**

- Title: What is Amazon Managed Service for Apache Flink?
- URL: https://docs.aws.amazon.com/managed-flink/latest/java/what-is.html
- Source type: AWS service documentation
- Relevant passage: "you can use Java, Scala, Python, or SQL to process and analyze streaming data." Studio notebooks allow interactive queries with "standard SQL, Python, and Scala".
- Publication or update date: Not shown
- Access status: page successfully inspected

**Evidence analysis**

Explicitly supports the broader language list. The "formerly: Amazon Kinesis Data Analytics" mapping is not stated on the inspected page, so it is neither confirmed nor contradicted here. "Fink" is a typo (style, not reported).

**Final technical assessment**

Technically correct but incomplete.

**Final exam-scope assessment**

Reasonably covered by a broader exam objective (Task Statement 3.5, streaming data services). Low–medium relevance.

**Recommended disposition**

EXPAND

**Proposed replacement text**

> Use Java, Scala, Python or SQL (Apache Flink)

**Rationale**

Replaces the single bullet; the two preceding bullets remain accurate.

**Correction priority:** LOW

**Next action:** Ready for correction

---

### FOUND-006 — CloudTrail Lake availability and Athena claim
**Original category:** OUTDATED
**Original triage confidence:** MEDIUM
**Verification outcome:** CONFIRMED
**Verified confidence:** HIGH

**Original note**

> If longer than 90 days needed, create a Trail which outputs to S3 and requires Amazon Athena to analyse efficiently (CloudTrail Lake uses Athena under the hood).

**Triage allegation**

(a) CloudTrail Lake has its own SQL engine; Athena is only an optional federation route. (b) CloudTrail Lake is closed to new customers from 31 May 2026.

**Official sources inspected**

- Title: Working with AWS CloudTrail Lake
- URL: https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-lake.html
- Source type: AWS service documentation
- Relevant passage: "AWS CloudTrail Lake will no longer be open to new customers starting May 31, 2026… Existing customers can continue to use the service as normal." "CloudTrail Lake lets you run SQL-based queries on your events… converts existing events in row-based JSON format to Apache ORC format." "You can federate an event data store… and run SQL queries on the event data using Amazon Athena."
- Publication or update date: Not shown
- Access status: page successfully inspected

- Title: CloudTrail Lake availability change
- URL: https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-lake-service-availability-change.html
- Source type: AWS service documentation
- Relevant passage: "AWS CloudTrail Lake will no longer be open to new customers starting on May 31, 2026… Only AWS CloudTrail Lake is no longer open to new customers. Your AWS CloudTrail Trails… are not affected." Recommends migrating to Amazon CloudWatch.
- Publication or update date: Not shown
- Access status: page successfully inspected

**Evidence analysis**

Both sub-claims are explicit. Lake queries events with its own SQL; Athena is an optional federated route, so "uses Athena under the hood" is unsupported. The closure date (31 May 2026) is before today, so Lake is now closed to new customers. The trail → S3 → Athena pattern and CloudTrail itself are unaffected. The 90-day Event History statement was not re-verified in this stage.

**Final technical assessment**

Parenthetical is technically wrong (Athena "under the hood") and outdated (availability). The rest of the sentence is correct.

**Final exam-scope assessment**

Reasonably covered by a broader exam objective. AWS CloudTrail is in the in-scope Management and Governance list; auditing/governance falls under Domain 1 data governance (Task Statement 1.3).

**Recommended disposition**

CORRECT

**Proposed replacement text**

> (CloudTrail Lake can query events directly using SQL, but is no longer open to new customers)

in place of "(CloudTrail Lake uses Athena under the hood)".

**Rationale**

Fixes the incorrect mechanism and records availability with a minimal edit.

**Correction priority:** MEDIUM

**Next action:** Ready for correction

---

### FOUND-007 — Composite alarm actions
**Original category:** ERROR
**Original triage confidence:** MEDIUM
**Verification outcome:** CONFIRMED
**Verified confidence:** HIGH

**Original note**

> Only action for composite alarms is to publish to an SNS topic.

**Triage allegation**

Composite alarms can also invoke Lambda and Systems Manager actions. The "not EC2/Auto Scaling" parenthetical needed API-reference checking.

**Official sources inspected**

- Title: PutCompositeAlarm (Amazon CloudWatch API Reference)
- URL: https://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/API_PutCompositeAlarm.html
- Source type: AWS service documentation
- Relevant passage: "Composite alarms can take the following actions: Notify Amazon SNS topics. Invoke Lambda functions. Create OpsItems in Systems Manager Ops Center. Create incidents in Systems Manager Incident Manager." The `AlarmActions` valid values list SNS, Lambda, Systems Manager and an Amazon Q Developer investigation action; EC2 and Auto Scaling action ARNs are not listed.
- Publication or update date: Not shown
- Access status: page successfully inspected

- Title: Create a composite alarm
- URL: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Create_Composite_Alarm.html
- Source type: AWS service documentation
- Relevant passage: "Configure actions" offers SNS notification, "Add Lambda action" and "Add Systems Manager action".
- Publication or update date: Not shown
- Access status: page successfully inspected

**Evidence analysis**

Two pages explicitly list more than SNS. That EC2/Auto Scaling actions are not available follows from their absence in the explicit valid-values list for the composite alarm API; I treat that as supported by the API reference but it is an inference from a list, not an explicit "not supported" statement.

**Final technical assessment**

Technically wrong as written (may have reflected launch behaviour).

**Final exam-scope assessment**

Reasonably covered by a broader exam objective (Amazon CloudWatch in the in-scope list). Low exam relevance; the key point is composite alarms reduce alarm noise.

**Recommended disposition**

CORRECT

**Proposed replacement text**

> Composite alarms can publish to an SNS topic, invoke a Lambda function, or take Systems Manager actions (OpsItems / Incident Manager). Unlike metric alarms, they cannot perform EC2 or Auto Scaling actions.

**Rationale**

Corrects the false "only SNS" claim. The EC2/Auto Scaling sentence can be dropped if the author prefers not to rely on an inference.

**Correction priority:** MEDIUM

**Next action:** Ready for correction

---

### FOUND-008 — AWS Audit Manager closed to new customers
**Original category:** OUTDATED
**Original triage confidence:** HIGH
**Verification outcome:** CONFIRMED
**Verified confidence:** HIGH

**Original note**

> For continually auditing AWS usage to simplify risk and assess compliance.

(entire "AWS Audit Manager" section, which presents the service as generally available)

**Triage allegation**

Audit Manager no longer accepts new customers; the note lacks this caveat. Exam-scope interpretation to be reviewed.

**Official sources inspected**

- Title: What is AWS Audit Manager?
- URL: https://docs.aws.amazon.com/audit-manager/latest/userguide/what-is.html
- Source type: AWS service documentation
- Relevant passage: "AWS Audit Manager is no longer open to new customers. Existing customers can continue to use the service as normal." Description of continual auditing and frameworks matches the note.
- Publication or update date: Not shown
- Access status: page successfully inspected

- Title: AWS Certified Solutions Architect – Associate (SAA-C03) Exam Guide v1.1
- URL: https://d1.awsstatic.com/training-and-certification/docs-sa-assoc/AWS-Certified-Solutions-Architect-Associate_Exam-Guide.pdf
- Source type: official exam guide
- Relevant passage: "In-scope AWS services and features" → Security, Identity, and Compliance lists "AWS Audit Manager".
- Publication or update date: Version 1.1, no date shown
- Access status: page successfully inspected (PDF text extracted)

**Evidence analysis**

The availability change is explicit. The note's technical content remains accurate. The exam guide explicitly lists Audit Manager as in scope, so closure to new customers is not evidence of removal from the exam; a newer guide could change this (not found).

**Final technical assessment**

Technically correct but outdated/incomplete (missing availability caveat).

**Final exam-scope assessment**

Explicitly covered by a current exam objective (in-scope services list, Security, Identity, and Compliance). Retain in the notes.

**Recommended disposition**

CLARIFY

**Proposed replacement text**

> Note: AWS Audit Manager is no longer open to new customers (existing customers can continue to use it).

Add as one line under the heading.

**Rationale**

Keeps the content (still in scope) and adds the one fact a learner needs.

**Correction priority:** LOW

**Next action:** Ready for correction

---

### FOUND-009 — Amazon Inspector description is the legacy model
**Original category:** OUTDATED
**Original triage confidence:** HIGH (current capabilities); LOW ("699 checks", "passive scans")
**Verification outcome:** PARTIALLY SUPPORTED
**Verified confidence:** HIGH (current-service description); LOW ("699 checks", "passive scans")

**Original note**

> AWS Inspector runs a security benchmark against specific EC2 instances. It can run a variety of security benchmarks and both Network and Host assessments.
>
> Steps: 1. Install AWS SSM agent on EC2 instance 2. Add relevant permissions 3. Run an assessment on the target 4. Review findings and remediate security issues found
>
> A popular benchmark is by CIS and has 699 checks.
>
> There are also passive scans.

**Triage allegation**

The note describes an assessment-run, benchmark-centred model; the current service continuously and automatically scans EC2, ECR images and Lambda for vulnerabilities and network exposure.

**Official sources inspected**

- Title: What is Amazon Inspector?
- URL: https://docs.aws.amazon.com/inspector/latest/user/what-is-inspector.html
- Source type: AWS service documentation
- Relevant passage: "Amazon Inspector is a vulnerability management service that automatically discovers workloads and continually scans them for software vulnerabilities and unintended network exposure… discovers and scans Amazon EC2 instances, container images in Amazon ECR, and Lambda functions." "you don't need to manually schedule or configure assessment scans."
- Publication or update date: Not shown
- Access status: page successfully inspected

- Title: Scanning Amazon EC2 instances with Amazon Inspector
- URL: https://docs.aws.amazon.com/inspector/latest/user/scanning-ec2.html
- Source type: AWS service documentation
- Relevant passage: "agent-based scanning collects software inventory… using the… SSM agent, and agentless scanning collects software inventory using Amazon EBS snapshots." Network reachability scans run "once every 12 hours".
- Publication or update date: Not shown
- Access status: page successfully inspected

- Title: Center for Internet Security (CIS) scans for Amazon EC2 instance operating systems
- URL: https://docs.aws.amazon.com/inspector/latest/user/scanning-cis.html
- Source type: AWS service documentation
- Relevant passage: "Amazon Inspector CIS scans (CIS scans) benchmark your Amazon EC2 instance operating systems…" scans are run or scheduled against tagged instances; the instance must be an SSM managed instance.
- Publication or update date: Not shown
- Access status: page successfully inspected

- Title: Amazon Inspector Classic user guide
- URL: https://docs.aws.amazon.com/inspector/v1/userguide/inspector_introduction.html
- Source type: AWS service documentation
- Relevant passage: none
- Publication or update date: n/a
- Access status: inaccessible (HTTP 404)

**Evidence analysis**

The core service description in the note is not the current primary model: Inspector now scans automatically and continuously across three resource types. However, Inspector still has a CIS benchmark scanning feature for EC2 OS configuration with SSM prerequisites, so the note is not entirely obsolete. Whether CIS scans have "699 checks" and what "passive scans" refers to are not covered by the inspected pages and remain unverified. Inspector Classic's retirement is not evidenced and is not claimed.

**Final technical assessment**

Technically correct for a legacy assessment-run model/benchmark feature, but outdated and incomplete as a description of Amazon Inspector. "699 checks" and "passive scans" cannot be reliably assessed.

**Final exam-scope assessment**

Explicitly covered by a current exam objective: Amazon Inspector is in the in-scope Security, Identity, and Compliance list (Task Statement 1.2 security services with appropriate use cases; Task Statement 1.3 compliance).

**Recommended disposition**

CORRECT

**Proposed replacement text**

Replace the paragraph beginning "AWS Inspector runs a security benchmark…" and the "Steps" list with:

> Amazon Inspector is a vulnerability management service that automatically and continually scans EC2 instances, ECR container images and Lambda functions for software vulnerabilities (CVEs) and unintended network exposure, and produces findings with a risk score. EC2 package scanning uses the SSM agent (agent-based) or EBS snapshots (agentless). Inspector can also run CIS benchmark scans against EC2 instance operating systems (hardening checks); these require SSM-managed instances.

Keep the "Hardening" paragraph. Remove "699 checks" and "passive scans" unless the author can source them.

**Rationale**

Corrects the main service description, which is a high-value SAA topic (Inspector vs GuardDuty vs Macie), while retaining the still-valid hardening/CIS idea.

**Correction priority:** HIGH

**Next action:** Ready for correction (for the proposed text; "699 checks" and "passive scans" remain unverified and are only being removed, not asserted false)

---

### FOUND-010 — Macie described as CloudTrail anomaly detection
**Original category:** ERROR
**Original triage confidence:** HIGH
**Verification outcome:** CONFIRMED
**Verified confidence:** HIGH

**Original note**

> Fully-managed service that continuously monitors S3 data access activity for anomalies and generates detailed alerts when it detects risk of unauthorised access or inadvertent data leaks.
> Works by using machine learning to analyse CloudTrail logs.
> Alerts: Anonymised access, Config compliance, Credential loss, … Suspicious access
>
> Macie will identify the users most at-risk of leading to a compromise.

**Triage allegation**

Macie is a sensitive data discovery and S3 security posture service, not a CloudTrail-based access-anomaly service.

**Official sources inspected**

- Title: What is Amazon Macie?
- URL: https://docs.aws.amazon.com/macie/latest/user/what-is-macie.html
- Source type: AWS service documentation
- Relevant passage: "Amazon Macie is a data security service that discovers sensitive data by using machine learning and pattern matching, provides visibility into data security risks, and enables automated protection against those risks." Macie "automatically evaluates and monitors the buckets for security and access control". Its Related services section attributes analysis of "AWS CloudTrail data event logs for Amazon S3 and CloudTrail management event logs" with threat intelligence to Amazon GuardDuty, not Macie.
- Publication or update date: Not shown
- Access status: page successfully inspected

- Title: Types of Macie findings
- URL: https://docs.aws.amazon.com/macie/latest/user/findings-types.html
- Source type: AWS service documentation
- Relevant passage: "Amazon Macie generates two categories of findings: policy findings and sensitive data findings." Policy types include `Policy:IAMUser/S3BlockPublicAccessDisabled`, `S3BucketPublic`, `S3BucketSharedExternally`; sensitive data types include `SensitiveData:S3Object/Financial`, `Credentials`, `Personal`. The page does not mention the note's alert names (e.g. Ransomware, Privilege escalation).
- Publication or update date: Not shown
- Access status: page successfully inspected

- Title: SAA-C03 Exam Guide v1.1
- URL: https://d1.awsstatic.com/training-and-certification/docs-sa-assoc/AWS-Certified-Solutions-Architect-Associate_Exam-Guide.pdf
- Source type: official exam guide
- Relevant passage: Task Statement 1.2 knowledge: "Security services with appropriate use cases (for example, Amazon Cognito, Amazon GuardDuty, Amazon Macie)"; Macie also in the in-scope services list.
- Publication or update date: Version 1.1, no date shown
- Access status: page successfully inspected (PDF text extracted)

**Evidence analysis**

The current purpose is explicit and different from the note. Absence of the legacy alert names on the findings page is not by itself proof they never existed, but the current finding model is explicit (policy and sensitive data findings), and the note's anomaly/CloudTrail description matches what AWS now assigns to GuardDuty. The "users most at-risk" sentence is not supported by any inspected source.

**Final technical assessment**

Technically wrong for the current service (original 2017-era description).

**Final exam-scope assessment**

Explicitly covered by a current exam objective (Task Statement 1.2; in-scope list). The Macie vs GuardDuty vs Inspector distinction is directly relevant.

**Recommended disposition**

CORRECT

**Proposed replacement text**

Replace the first paragraph, the "Alerts" list and the final sentence with:

> Fully-managed data security service that uses machine learning and pattern matching to discover sensitive data (e.g. PII) in Amazon S3, and evaluates and monitors S3 buckets for security and access control (e.g. publicly accessible buckets).
> Generates two categories of findings: policy findings (bucket security/access issues) and sensitive data findings (sensitive data found in objects). Findings can go to EventBridge and Security Hub.

**Rationale**

The existing text would lead to wrong answers in service-selection questions. Replacing the whole section is justified because every sentence is affected.

**Correction priority:** HIGH

**Next action:** Ready for correction

---

### FOUND-011 — Security Hub naming
**Original category:** OUTDATED
**Original triage confidence:** MEDIUM
**Verification outcome:** CONFIRMED
**Verified confidence:** MEDIUM

**Original note**

> Cloud Security Posture Management (CSPM), allowing generation of a security score to determine security posture. Allows enabling of standards \= collections of security controls \= AWS Config rules.

**Triage allegation**

The described capability is now documented as "AWS Security Hub CSPM". "Controls = AWS Config rules" was unverified.

**Official sources inspected**

- Title: Introduction to AWS Security Hub CSPM
- URL: https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html
- Source type: AWS service documentation
- Relevant passage: "AWS Security Hub Cloud Security Posture Management (AWS Security Hub CSPM) provides you with a comprehensive view of your security state in AWS…"; standards include AWS FSBP, CIS, PCI DSS, NIST; "Each standard includes several security controls"; security scores are calculated.
- Publication or update date: Not shown
- Access status: page successfully inspected

- Title: Enabling and configuring AWS Config for Security Hub CSPM
- URL: https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-setup-prereqs.html
- Source type: AWS service documentation
- Relevant passage: "AWS Security Hub CSPM uses AWS Config rules to run security checks and generate findings for most controls."
- Publication or update date: Not shown
- Access status: page successfully inspected

- Title: AWS Security Hub Documentation (landing page)
- URL: https://docs.aws.amazon.com/securityhub/
- Source type: AWS service documentation
- Relevant passage: User guide "describes how to set up, configure, and use AWS Security Hub and Security Hub CSPM".
- Publication or update date: Not shown
- Access status: partial access (no detail on how the two relate)

**Evidence analysis**

The CSPM documentation name is explicit, and the note's description (standards, controls, security score) matches it. "Controls = AWS Config rules" is a simplification: AWS states Config rules are used for "most" controls, not all. Docs show both "AWS Security Hub" and "AWS Security Hub CSPM" now exist; how they relate was not established, so no claim is made about the broader Security Hub. The exam guide lists "AWS Security Hub".

**Final technical assessment**

Technically correct but outdated (naming) and slightly imprecise ("= AWS Config rules").

**Final exam-scope assessment**

Explicitly covered by a current exam objective (in-scope Security, Identity, and Compliance list lists AWS Security Hub).

**Recommended disposition**

CLARIFY

**Proposed replacement text**

> AWS Security Hub CSPM: Cloud Security Posture Management (CSPM), allowing generation of a security score to determine security posture. Allows enabling of standards \= collections of security controls (most evaluated using AWS Config rules).

**Rationale**

Aligns with current documentation naming without asserting how it relates to the newer Security Hub.

**Correction priority:** LOW

**Next action:** Ready for correction

---

### FOUND-012 — CloudWatch Logs Insights limits
**Original category:** OUTDATED
**Original triage confidence:** LOW
**Verification outcome:** CONFIRMED
**Verified confidence:** MEDIUM

**Original note**

> Single request can query up to 20 log groups, times out after 15 mins and results are available for 7 days.

**Triage allegation**

Limits may have changed; the triage stage could not verify the values.

**Official sources inspected**

- Title: Analyzing log data with CloudWatch Logs Insights
- URL: https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AnalyzingLogData.html
- Source type: AWS service documentation
- Relevant passage: "Queries using any of the supported query languages time out after 60 minutes, if they have not completed. Query results are available for 7 days." Also lists three query languages (Logs Insights QL, OpenSearch PPL, OpenSearch SQL).
- Publication or update date: Not shown
- Access status: page successfully inspected

- Title: StartQuery (Amazon CloudWatch Logs API Reference)
- URL: https://docs.aws.amazon.com/AmazonCloudWatchLogs/latest/APIReference/API_StartQuery.html
- Source type: AWS service documentation
- Relevant passage: `logGroupIdentifiers` / `logGroupNames`: "You can include up to 50 log groups."
- Publication or update date: Not shown
- Access status: page successfully inspected

- Title: Supported logs and discovered fields
- URL: https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CWL_AnalyzeLogData-discoverable-fields.html
- Source type: AWS service documentation
- Relevant passage: "CloudWatch Logs Insights supports different log types"; "@" system fields are generated for log groups in the Standard class; field discovery is supported only for the Standard log class.
- Publication or update date: Not shown
- Access status: page successfully inspected

**Evidence analysis**

The 15-minute timeout is directly contradicted (60 minutes). The 20-log-group figure is contradicted by the API reference (up to 50 listed per request); a separate console limit was not checked. The 7-day retention of results is confirmed. "Supports all types of logs" is neither confirmed nor contradicted: the docs state support for different log types, with field discovery limited to Standard class log groups.

**Final technical assessment**

Partly outdated: two of the three figures (20 log groups, 15 minutes) are no longer current; 7 days is correct. Wording on "all types of logs" cannot be reliably assessed.

**Final exam-scope assessment**

Reasonably covered by a broader exam objective (Amazon CloudWatch in-scope list). Exact numbers are unlikely to be tested.

**Recommended disposition**

CORRECT

**Proposed replacement text**

> Single request can query up to 50 log groups, times out after 60 mins and results are available for 7 days.

**Rationale**

Minimal numeric correction. The author may instead delete the figures as low-value.

**Correction priority:** LOW

**Next action:** Ready for correction

---

### FOUND-013 — Elasticsearch Service references
**Original category:** OUTDATED
**Original triage confidence:** MEDIUM
**Verification outcome:** PARTIALLY SUPPORTED
**Verified confidence:** MEDIUM

**Original note**

> - Stream to ElasticSearch Service (ES) \- to use ELK stack

and

> Two engines can be deployed:
> - OpenSearch … Open source fork of ElasticSearch and Kibana
> - ElasticSearch … Search engine based on Lucene library … Free and open (for non-enterprise?)

**Triage allegation**

The service is Amazon OpenSearch Service; Elasticsearch is only a legacy engine; the licensing "?" is unresolved.

**Official sources inspected**

- Title: What is Amazon OpenSearch Service?
- URL: https://docs.aws.amazon.com/opensearch-service/latest/developerguide/what-is.html
- Source type: AWS service documentation
- Relevant passage: "Amazon OpenSearch Service supports OpenSearch and legacy Elasticsearch OSS (up to 7.10, the final open source version of the software). When you create a domain, you have the option of which search engine to use."
- Publication or update date: Not shown
- Access status: page successfully inspected

- Title: Streaming CloudWatch Logs data to Amazon OpenSearch Service
- URL: https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CWL_OpenSearch_Stream.html
- Source type: AWS service documentation
- Relevant passage: "You can configure a log group in Amazon CloudWatch Logs, so you can stream data to your Amazon OpenSearch Service cluster in near real-time."
- Publication or update date: Not shown
- Access status: page successfully inspected

**Evidence analysis**

The CloudWatch Logs line uses an outdated service name: current docs say "Amazon OpenSearch Service". The "Two engines can be deployed" statement is still consistent with the docs, which say both OpenSearch and legacy Elasticsearch OSS can be chosen. The change needed there is the "legacy" qualifier and version limit, not removal. The docs also describe Elasticsearch OSS up to 7.10 as "the final open source version", which addresses the author's "?" licensing question, though the note's wording "(for non-enterprise?)" is the author's own question, not a factual claim. The triage statement that the page lists OpenSearch Serverless as a deployment option was not confirmed on the inspected page (only a Serverless usage-type code was seen); the note already mentions Serverless.

**Final technical assessment**

CloudWatch Logs line: technically correct but outdated (name). OpenSearch section: technically correct but incomplete/ambiguous (Elasticsearch is legacy, OSS up to 7.10).

**Final exam-scope assessment**

Explicitly covered by a current exam objective: Amazon OpenSearch Service is in the in-scope Analytics list. Elasticsearch as a separate service is not.

**Recommended disposition**

CLARIFY

**Proposed replacement text**

> - Stream to OpenSearch Service \- to use ELK stack

> - ElasticSearch (legacy; OSS versions up to 7.10)
>   - Search engine based on Lucene library
>   - Free and open (7.10 is the final open source version)

**Rationale**

Updates the name and labels Elasticsearch as legacy while preserving the author's structure; also resolves the unanswered "?" using the docs.

**Correction priority:** MEDIUM

**Next action:** Ready for correction

---

### FOUND-014 — Core SAA-C03 analytics services not in this section
**Original category:** INCOMPLETE
**Original triage confidence:** LOW
**Verification outcome:** PARTIALLY SUPPORTED
**Verified confidence:** MEDIUM

**Original note**

N/A (omission). The section covers CloudWatch, EventBridge, Kinesis, CloudTrail, tracing/AMP, security services and OpenSearch.

**Triage allegation**

The exam guide lists Athena, Lake Formation, QuickSight, Glue, EMR, MSK and Redshift, which this section does not cover in depth. Coverage elsewhere in the notes was not checked.

**Official sources inspected**

- Title: AWS Certified Solutions Architect – Associate (SAA-C03) Exam Guide v1.1
- URL: https://d1.awsstatic.com/training-and-certification/docs-sa-assoc/AWS-Certified-Solutions-Architect-Associate_Exam-Guide.pdf
- Source type: official exam guide
- Relevant passage: Task Statement 3.5 knowledge: "Data analytics and visualization services… (for example, Amazon Athena, AWS Lake Formation, Amazon QuickSight)", "Data transformation services… (for example, AWS Glue)", "Streaming data services… (for example, Amazon Kinesis)"; skills: "Selecting appropriate compute options for data processing (for example, Amazon EMR)". In-scope Analytics list: Athena, Data Exchange, Data Pipeline, EMR, Glue, Kinesis, Lake Formation, MSK, OpenSearch Service, QuickSight, Redshift.
- Publication or update date: Version 1.1, no date shown
- Access status: page successfully inspected (PDF text extracted)

Local check of other notes (`notes/working/`):
- `dbs.md` contains sections for Amazon Redshift, Amazon Athena, AWS Glue (jobs, Studio, Data Catalogue), AWS Lake Formation, AWS Data Exchange, with QuickSight and EMR mentioned only as integrations.
- No dedicated coverage of Amazon EMR, Amazon QuickSight or Amazon MSK was found in the working notes by keyword search.

**Evidence analysis**

The exam guide content is confirmed. Most of the named services are covered elsewhere in the notes, so the whole-section omission claim is largely not supported. The gap that remains is dedicated coverage of EMR, QuickSight and MSK (and possibly Data Pipeline); a keyword search does not prove depth of coverage, and whether these should be added is a content-scope decision rather than an error in existing notes.

**Final technical assessment**

Not an error in existing text. The notes have a possible coverage gap for EMR, QuickSight and MSK that cannot be fully assessed without reviewing the notes' depth.

**Final exam-scope assessment**

Explicitly covered by a current exam objective (Task Statement 3.5; in-scope Analytics list).

**Recommended disposition**

INVESTIGATE_FURTHER

**Proposed replacement text**

No replacement proposed pending further verification.

**Rationale**

The finding should not drive a correction in `analytics.md`. Recommend a separate coverage decision on EMR, QuickSight and MSK, kept at SAA level.

**Correction priority:** LOW

**Next action:** Further research required

---

## 3. Correction-ready summary

| Finding ID | Outcome | Disposition | Correction priority | Ready for correction? |
|---|---|---|---|---|
| FOUND-001 | CONFIRMED | CORRECT | LOW | Yes |
| FOUND-002 | CONFIRMED | CLARIFY | LOW | Yes |
| FOUND-003 | UNRESOLVED | INVESTIGATE_FURTHER | LOW | No |
| FOUND-004 | CONFIRMED | CORRECT | MEDIUM | Yes |
| FOUND-005 | CONFIRMED | EXPAND | LOW | Yes |
| FOUND-006 | CONFIRMED | CORRECT | MEDIUM | Yes |
| FOUND-007 | CONFIRMED | CORRECT | MEDIUM | Yes |
| FOUND-008 | CONFIRMED | CLARIFY | LOW | Yes |
| FOUND-009 | PARTIALLY SUPPORTED | CORRECT | HIGH | Yes (main description only; "699 checks"/"passive scans" unverified) |
| FOUND-010 | CONFIRMED | CORRECT | HIGH | Yes |
| FOUND-011 | CONFIRMED | CLARIFY | LOW | Yes |
| FOUND-012 | CONFIRMED | CORRECT | LOW | Yes |
| FOUND-013 | PARTIALLY SUPPORTED | CLARIFY | MEDIUM | Yes |
| FOUND-014 | PARTIALLY SUPPORTED | INVESTIGATE_FURTHER | LOW | No |
