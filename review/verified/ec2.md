# Verification Report: EC2

- **Notes section:** `notes/working/ec2.md`
- **Triage report:** `review/findings/ec2.md`
- **Verification date:** 2026-10-09
- **Exam scope authority:**
  - Current SAA-C03 exam guide on docs.aws.amazon.com (domain pages and in-scope/out-of-scope lists): https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/solutions-architect-associate-03.html
  - SAA-C03 exam guide PDF, "Version 1.1" (no date shown), downloaded and text-extracted: https://d1.awsstatic.com/training-and-certification/docs-sa-assoc/AWS-Certified-Solutions-Architect-Associate_Exam-Guide.pdf

## 1. Summary table

| Finding ID | Verification outcome | Recommended disposition | Priority | Change since triage? |
| --- | --- | --- | --- | --- |
| FOUND-001 | CONFIRMED | CORRECT | LOW | NO |
| FOUND-002 | CONFIRMED | CORRECT | LOW | NO |
| FOUND-003 | CONFIRMED | CORRECT | MEDIUM | NO |
| FOUND-004 | CONFIRMED | CORRECT | HIGH | NO |
| FOUND-005 | CONFIRMED | CORRECT | MEDIUM | NO |
| FOUND-006 | CONFIRMED | CORRECT | LOW | NO |
| FOUND-007 | CONFIRMED | CORRECT | LOW | NO |
| FOUND-008 | CONFIRMED | CORRECT | LOW | YES |
| FOUND-009 | CONFIRMED | CORRECT | LOW | NO |
| FOUND-010 | CONFIRMED | EXPAND | HIGH | YES |
| FOUND-011 | CONFIRMED | CORRECT | MEDIUM | NO |
| FOUND-012 | CONFIRMED | CORRECT | MEDIUM | YES |
| FOUND-013 | CONFIRMED | EXPAND | LOW | NO |
| FOUND-014 | PARTIALLY SUPPORTED | CLARIFY | LOW | YES |
| FOUND-015 | PARTIALLY SUPPORTED | CLARIFY | MEDIUM | YES |
| FOUND-016 | CONFIRMED | CORRECT | LOW | YES |

No triage finding was rejected, so there are no false-positive entries.

## 2. Details for changed findings

### FOUND-008 — EC2-Classic / Scheduled RI references
**Change from triage:** The retirement is now stated as complete, not merely "expected". The size-change qualifier is more precise than triage proposed.
**Original assessment:** CONFIRMED, MEDIUM. EC2-Classic retired ("we expect… 15 Aug 2022"); the network-platform modification is obsolete.
**Verified conclusion:** Technically outdated. The "Change network EC2-classic -> VPC and vice versa" bullet should be deleted. The note's "Change instance size" bullet is correct only for Linux/UNIX and within the same instance family and generation.
**Official evidence:** The AWS blog carries the update "(August 23, 2023) – The retirement announced in this blog post is now complete. There are no more EC2 instances running with EC2-Classic networking." The Modify Reserved Instances page lists three modifiable attributes: Availability Zone (Linux and Windows), scope (Linux and Windows), and instance size "within the same instance family and generation" (Linux/UNIX only). No network-platform change is listed.
**Source:**
- EC2-Classic Networking is Retiring – AWS News Blog – https://aws.amazon.com/blogs/aws/ec2-classic-is-retiring-heres-how-to-prepare/ (update note at top)
- Modify Reserved Instances – https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ri-modifying.html (modifiable attributes table)

**Recommended disposition:** CORRECT
**Proposed replacement:** Delete the EC2-Classic bullet. Reword "Change instance size" as "Change instance size (same instance family and generation; Linux/UNIX only)".
**Remaining uncertainty:** None.

### FOUND-010 — Hibernation not covered
**Change from triage:** Triage's open "verify prerequisites/limits" item is now resolved, so a concrete replacement can be given. Triage cited the wrong task statement.
**Original assessment:** CONFIRMED, HIGH. Hibernate action and state missing; details to be verified before writing.
**Verified conclusion:** Technically correct but incomplete. The notes' action list (Launch/Stop/Start/Terminate/Reboot/Retire/Recover) has no Hibernate. Hibernation is named in exam scope under Task 4.2 "Design cost-optimized compute" (knowledge: "Scaling strategies (for example, auto scaling, hibernation)"; skills: "EC2 hibernation"), not Task 3.2.
**Official evidence:**
- Hibernation saves RAM to the EBS root volume, EBS volumes persist, and on start the RAM is reloaded, processes resume and the instance keeps its instance ID.
- No instance-usage charge while hibernated. You still pay for EBS storage, including the storage holding the RAM contents.
- Hibernation must be enabled at launch, because it "can't [be enabled] on an existing instance, whether it is running or stopped."
- The root volume must be an encrypted EBS volume (not instance store) and large enough to hold RAM.
- RAM must be under 150 GiB for Linux instances (16 GiB or less for Windows).
- A hibernation-capable AMI is required.

**Source:**
- Hibernate your Amazon EC2 instance – https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Hibernate.html
- Prerequisites for EC2 instance hibernation – https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/hibernating-prerequisites.html
- Content Domain 4 (Task 4.2) – https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/solutions-architect-associate-03-domain4.html
- Exam guide PDF v1.1, domain 4 (hibernation appears twice)

**Recommended disposition:** EXPAND
**Proposed replacement:** Add an action: "Hibernate – saves RAM contents to the (encrypted) EBS root volume, then stops the instance. On start, RAM is reloaded and processes resume (fast warm-up). Must be enabled at launch; no instance charge while hibernated, but EBS storage is still charged." Add the corresponding state if the author lists states in full.
**Remaining uncertainty:** Supported instance families and the exact RAM limit may vary with instance family. Only the Linux 150 GiB and Windows 16 GiB figures were read, so they are left out of the replacement.

### FOUND-012 — Service Connect "evolution of App Mesh"
**Change from triage:** The exam-scope sub-claim is only partly supported by current sources.
**Original assessment:** CONFIRMED, HIGH. App Mesh support ends 2026-09-30; App Mesh is on the exam guide's out-of-scope list.
**Verified conclusion:** Technically outdated, as triage said.
- **App Mesh:** the AWS notice supports the end-of-support point. The notice is still worded in the future tense ("will discontinue support… After September 30, 2026, you will no longer be able to access…") even though the date has passed. The page being reachable means actual shutdown could not be confirmed.
- **Exam scope:** the v1.1 PDF lists AWS App Mesh and AWS Cloud Map as out-of-scope. The current docs-site out-of-scope list no longer contains App Mesh, but still contains AWS Cloud Map. App Mesh appears on neither the in-scope nor the out-of-scope docs list. This is consistent with removal after discontinuation, but that is my inference and is not stated by AWS.
- **Cloud Map:** the note says Service Connect abstracts "App Mesh, Cloud Map and ELB" configuration, and Cloud Map is out of scope in both sources.

**Official evidence:** "End of support notice: On September 30, 2026, AWS will discontinue support for AWS App Mesh." The page points to "Migrating from AWS App Mesh to Amazon ECS Service Connect."
**Source:**
- What Is AWS App Mesh? – https://docs.aws.amazon.com/app-mesh/latest/userguide/what-is-app-mesh.html (end-of-support notice)
- Out-of-Scope AWS Services – https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/saa-03-out-of-scope-services.html
- Exam guide PDF v1.1 (out-of-scope list: AWS App Mesh, AWS Cloud Map)

**Recommended disposition:** CORRECT
**Proposed replacement:** "Successor to AWS App Mesh (AWS announced end of support on 30 September 2026), abstracting much of the configuration between Cloud Map and ELB."
**Remaining uncertainty:** Actual post-30-September-2026 service status was not confirmed. Do not state that App Mesh is out of scope on the strength of the current docs page alone.

### FOUND-014 — ENA "up to 100Gb/s"
**Change from triage:** UNVERIFIED candidate → PARTIALLY SUPPORTED.
**Original assessment:** OUTDATED candidate, LOW confidence; not checked.
**Verified conclusion:** The "100Gb/s" ceiling is no longer accurate as a maximum, but the evidence is indirect. The EC2 instance-type specification pages list network bandwidth well above 100 Gbps for the largest current instance types. Examples include 200 Gbps (e.g. `m6in.32xlarge`), 400 Gbps and 600 Gbps (e.g. `m8in.96xlarge`), which require multiple network cards and ENIs. The same page refers to "configurable ENA queue allocation", which ties these instances to ENA.
**Official evidence:** The general-purpose instance types page lists throughput up to 600 Gbps for `m8in.96xlarge` (two ENIs on separate network cards at up to 300 Gbps each). The ENA enablement page states no bandwidth figure.
**Source:**
- Amazon EC2 general purpose instance types – https://docs.aws.amazon.com/ec2/latest/instancetypes/gp.html (network performance footnotes)
- Enable enhanced networking with ENA – https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/enhanced-networking-ena.html

**Recommended disposition:** CLARIFY
**Proposed replacement:** "…ENA enabled, giving high-bandwidth networking (100+ Gbps on larger instance types; actual bandwidth depends on instance type)." Low exam relevance, so the author may just remove the figure.
**Remaining uncertainty:** No official page states "ENA supports up to N Gbps". Whether the 100 Gbps figure was ever an ENA-specific limit was not established.

### FOUND-015 — Dedicated Hosts vs Dedicated Instances
**Change from triage:** UNVERIFIED candidate → PARTIALLY SUPPORTED.
**Original assessment:** INCOMPLETE candidate, LOW confidence. Notes don't distinguish Dedicated Instances from Dedicated Hosts, and the purchase-option bullet may only apply to Instances.
**Verified conclusion:** Technically correct but incomplete.
- Dedicated Instances run on hardware dedicated to one account, with no visibility or control over placement, no host affinity (a stop/start may land on a different host), and limited BYOL.
- Dedicated Hosts are a physical server dedicated to you, with visibility and control over placement, host affinity and full per-socket/per-core/per-VM BYOL support.
- Dedicated Instances can use Dedicated Reserved Instances, Capacity Reservations and Dedicated Spot Instances.
- Dedicated Hosts have their own discount mechanism: Dedicated Host Reservations, "up to 70 percent compared to On-Demand Dedicated Host pricing", which require an allocated host.
- The page does not state that Spot is unavailable for Dedicated Hosts, so that is not asserted.
- The note's "Can be on-demand, reserved or spot" is true for Dedicated Instances but cannot be said of Hosts from the sources inspected.

**Official evidence:** Both pages compare the two options ("consider using a Dedicated Host instead" / "consider using Dedicated Instances instead").
**Source:**
- Amazon EC2 Dedicated Hosts – https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/dedicated-hosts-overview.html
- Amazon EC2 Dedicated Instances – https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/dedicated-instance.html

**Recommended disposition:** CLARIFY
**Proposed replacement:** Add under Dedicated: "Dedicated Instances – hardware dedicated to your account, no control of placement. Dedicated Hosts – a whole physical server with visibility/host affinity; needed for per-socket/per-core licence (BYOL) requirements. 'On-demand, reserved or spot' applies to Dedicated Instances."
**Remaining uncertainty:** On-demand per-host billing details and Spot for Hosts were not read.

### FOUND-016 — Elastic Beanstalk "Not recommended for Production"
**Change from triage:** UNVERIFIED candidate → CONFIRMED.
**Original assessment:** AMBIGUOUS candidate, LOW confidence; AWS documents production use.
**Verified conclusion:** Technically wrong as stated. AWS documents Beanstalk for production use.
**Official evidence:** "You can create and manage separate environments for development, testing, and production use, and you can deploy any version of your application to any environment." The Beanstalk overview describes managed provisioning of load balancing, health monitoring and auto scaling. The Beanstalk FAQ has an entry on "moving from test to production".
**Source:**
- Managing environments – https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.managing.html
- What is AWS Elastic Beanstalk? – https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/Welcome.html
- Elastic Beanstalk FAQs – https://aws.amazon.com/elasticbeanstalk/faqs/

**Recommended disposition:** CORRECT
**Proposed replacement:** Delete "Not recommended for Production applications." If the author's intent was that Beanstalk gives limited control compared with direct infrastructure management, replace with: "Less control over underlying infrastructure than managing EC2/ECS directly."
**Remaining uncertainty:** The author's original intent is unknown, and no official statement discouraging production use was found.

## 3. Verification notes on unchanged findings

These are unchanged from triage. They list only what was re-inspected.

- **FOUND-001:** The Hostname types page gives the other-region resource-name format as `ec2-instance-id.region.compute.internal`. The note's example is right and its placeholder is wrong. The IP-name line above it (`private-ipv4-address.region.compute.internal`) is correct.
- **FOUND-002:** The IMDS retrieval page uses `http://[fd00:ec2::254]/latest/meta-data/`.
- **FOUND-003:** An IAM role can be attached to or detached from a running or stopped instance, and the instance must have a role replaced (not attached again) if it already has one. No reboot requirement appears on the attach page. The IAM roles page says: "replace its instance profile. We do not recommend removing a role from an instance profile, because there is a delay of up to one hour".
- **FOUND-004:** Spread: "maximum of seven running instances in each Availability Zone" and can span AZs. Partition: "maximum of seven partitions per Availability Zone", and can span AZs. Cluster can't span AZs.
- **FOUND-005:** The Amazon Linux 2 FAQ states "Amazon Linux 2 reached its end of support on 2026-06-30." The Amazon Linux AMI FAQ states AL1 "reached its end of life on December 31, 2023."
- **FOUND-006:**
  - SELinux is "enabled and set to permissive mode" (AL2023 SELinux page).
  - AL2023 "includes components sourced from multiple versions of Fedora and other distributions, such as CentOS 9 Stream."
  - "`cronie` is not included by default" (confirmed on the AL2023 `systemd` timers page).
  - Also confirmed on the AL2-vs-AL2023 comparison page: gp3 default volume, Corretto default JVM.
- **FOUND-007:** The RI page says "up to 72%". The RI pricing page says Convertible RIs give "up to 66%".
- **FOUND-009:** The RI Marketplace page says selling limits "apply to the lifetime of your AWS account… not annual limits and they can't be increased. You can sell up to $50,000 in Reserved Instances." It also confirms 30 days active, the US-address bank requirement, and Standard-only. The note's "Instances in GovCloud region cannot be sold" is not on the page, which lists instead "Region that is disabled by default" and an AWS India restriction. It could not be confirmed or refuted.
- **FOUND-011:** The ECS task definition parameters page says "For Amazon ECS tasks hosted on Fargate, the `awsvpc` network mode is required." This is the explicit statement triage could not quote. `awslogs` is a log driver.
- **FOUND-013:** The Compute Optimizer docs list supported resources and states a 14-day CloudWatch lookback, extendable to 93 days with enhanced infrastructure metrics. It also states it identifies idle resources. The supported-resource list is now much longer than the note's list. Besides the RDS/Aurora and "commercial software licenses" (which the note shows as SQL Server licenses) that triage mentioned, it includes NAT Gateway, DynamoDB, ElastiCache, MemoryDB, DocumentDB, WorkSpaces and SageMaker. Any addition is optional at SAA level.

## 4. Totals and limitations

- **Findings reviewed:** 16
- **CONFIRMED:** 14 (001, 002, 003, 004, 005, 006, 007, 008, 009, 010, 011, 012, 013, 016)
- **PARTIALLY SUPPORTED:** 2 (014, 015)
- **REJECTED:** 0
- **UNRESOLVED:** 0
- **Materially changed since triage:** 6 (008, 010, 012, 014, 015, 016)

**Live-verification limitations**
- All pages above were fetched live on 2026-10-09 (docs pages in HTML or their `.md` renderings). AWS docs pages show no publication dates.
- The exam guide PDF is "Version 1.1" with no date. The docs-site exam guide pages are treated as the current authority for scope. The two differ at least on the App Mesh out-of-scope entry (FOUND-012), and a newer PDF version was not looked for.
- AWS App Mesh's actual shutdown after 2026-09-30 was not independently confirmed.
- FOUND-009: the note's GovCloud exclusion is neither confirmed nor refuted, and the "term rounded down to nearest month" and "seller can only set upfront price" bullets were not checked.
- FOUND-014: no official source states a per-ENA bandwidth ceiling.
- Triage's spot-checked "no finding" items (ASG predictive scaling, ECS Exec, ECS Anywhere pricing, Savings Plans percentages, Spot "up to 90%", RI limits, IMDSv2 behaviour) and the ECR, EKS and ELB sections were not re-verified here and are not endorsed.
