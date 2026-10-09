# Triage: EC2 (notes/working/ec2.md)

## Summary

- Section reviewed: `notes/working/ec2.md` (812 lines; EC2, ASG, ELB, ECR, ECS, Fargate, EKS, Beanstalk, Compute Optimizer)
- Official exam guide checked: AWS Certified Solutions Architect – Associate (SAA-C03) Exam Guide, **Version 1.1** (https://d1.awsstatic.com/training-and-certification/docs-sa-assoc/AWS-Certified-Solutions-Architect-Associate_Exam-Guide.pdf; downloaded and read). The cert page (https://aws.amazon.com/certification/certified-solutions-architect-associate/) was checked and still lists SAA-C03.
- Live verification status: Live web access available. Official AWS docs/FAQ/pricing pages were fetched (docs via their `.md` renderings) and inspected for the findings below.
- Findings: 13 CONFIRMED (ERROR 4, OUTDATED 5, INCOMPLETE 2, AMBIGUOUS 2) and 3 UNVERIFIED (candidates only). 0 UNRESOLVED.
- Important limitations:
  - The exam guide's date was not extractable from the PDF text; only "Version 1.1" is stated.
  - Not every sentence was researched. Claims that looked correct and were spot-checked (no finding): ASG predictive scaling (24h / 14 days / 48h / 6h), termination policy via Lambda, ECS Exec 20-min idle timeout and root, ECS Anywhere $0.01025/h, RI limits (20/month), RI Marketplace 30-day/US-bank rules, Compute Savings Plan 66%, EC2 Instance SP 72%, SageMaker SP 64%, Spot up to 90%, 169.254.169.254 / IMDSv2 401 behaviour.
  - Sections ECR, EKS, Beanstalk, ELB were not examined in depth.

## Findings

### FOUND-001 — Resource-name hostname for other regions uses wrong format
- **Category:** ERROR
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Hostnames → Resource Name → Other regions
- **Original claim:** "**Other regions** = [private-ipv4-address].region.compute.internal (e.g. i-0123456789abcdef.ca-central-1.compute.internal)"
- **Issue:** The format placeholder is the IPv4 address, but the example (and AWS format) uses the instance ID.
- **Official evidence:** Hostname types page: "Format for an instance in any other AWS Region: `ec2-instance-id.region.compute.internal`; Example: `i-0123456789abcdef.us-west-2.compute.internal`".
- **Source:** Hostname types – Amazon EC2 User Guide, https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/hostname-types.html
- **Assessment:** Note's example is right; its format string is a copy/paste error.
- **Suggested correction:** `[ec2-instance-id].region.compute.internal`
- **SAA-C03 relevance:** LOW
- **Further action:** None

### FOUND-002 — IMDS IPv6 address typo
- **Category:** ERROR
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Cloud-Init → Metadata → IPv6
- **Original claim:** "http://[fd00:ec2::245]/latest/meta-data/"
- **Issue:** Address is `fd00:ec2::254`, not `::245`.
- **Official evidence:** "…use the IPv6 address of the IMDS `[fd00:ec2::254]`… The instance must be a Nitro-based instance…"
- **Source:** Retrieve instance metadata – Amazon EC2 User Guide, https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html
- **Assessment:** Directly contradicted.
- **Suggested correction:** Change `245` to `254`.
- **SAA-C03 relevance:** LOW
- **Further action:** None

### FOUND-003 — Instance profile: reboot claim is incorrect
- **Category:** ERROR
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Instance Profile → bullets under "Can be associated at time of launch or on a running instance"
- **Original claim:** "Hard reboot is required if none were attached prior"; "Dissociating & reassociating or hard reboot will work"
- **Issue:** A role can be attached to a running or stopped instance with no reboot. To change roles, replace the instance profile; removing a role has an up-to-one-hour delay.
- **Official evidence:** "You can attach an IAM role to an instance that is running or stopped." / "To update permissions for an instance, replace its instance profile. We do not recommend removing a role from an instance profile, because there is a delay of up to one hour before this change takes effect."
- **Source:** Attach an IAM role to an instance (https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/attach-iam-role.html); IAM roles for Amazon EC2 (https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)
- **Assessment:** No reboot requirement appears in AWS guidance; "not instant" is consistent with the documented delay (my interpretation: the reboot workaround is unsupported).
- **Suggested correction:** Remove the reboot bullet(s); state that a role can be attached to a running/stopped instance without reboot, and prefer *replacing* the profile (removal can take up to 1 hour to take effect).
- **SAA-C03 relevance:** MEDIUM (IAM roles for EC2 is core)
- **Further action:** None

### FOUND-004 — Placement group details incomplete/misleading
- **Category:** AMBIGUOUS
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Placement Groups → Spread / Partition
- **Original claim:** "Max of **7** instances" (Spread); Partition has no limits/AZ info
- **Issue:** Spread limit is 7 *running instances per AZ per group* (hence a multi-AZ group can hold more). Partition groups also can span AZs, max 7 partitions per AZ.
- **Official evidence:** "A rack-level spread placement group can span multiple Availability Zones in the same Region, but it supports a maximum of seven running instances in each Availability Zone." / "A partition placement group can have partitions in multiple Availability Zones… maximum of seven partitions per Availability Zone." / "A cluster placement group can't span multiple Availability Zones."
- **Source:** Placement strategies – Amazon EC2 User Guide, https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-strategies.html
- **Assessment:** Note's Cluster and "multi-AZ" statements are correct; the limit wording is imprecise.
- **Suggested correction:** "Max of 7 running instances **per AZ**"; add to Partition: "Max 7 partitions per AZ; can be multi-AZ".
- **SAA-C03 relevance:** HIGH (classic exam topic)
- **Further action:** None

### FOUND-005 — Amazon Linux 1/2 end-of-life status
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Amazon Linux → Versions
- **Original claim:** "Amazon Linux 2 (AL2) - deprecated <check if deprecation went ahead and if AL2 specific content relevant now>"
- **Issue:** The author's TODO is resolved: AL2 reached end of support on 2026-06-30; AL1 reached EOL on 2023-12-31.
- **Official evidence:** AL2 FAQ: "Amazon Linux 2 reached its end of support on 2026-06-30." and "AL2 … is no longer receiving standard security updates." AL1 FAQ: "reached its end of life on December 31, 2023."
- **Source:** Amazon Linux 2 FAQs, https://aws.amazon.com/amazon-linux-2/faqs/ ; Amazon Linux AMI FAQs, https://aws.amazon.com/amazon-linux-ami/faqs/
- **Assessment:** Direct statements. AL2-specific content (e.g. yum) is now only legacy context.
- **Suggested correction:** Replace "deprecated" with "end of support 2026-06-30 (AL2) / end of life 2023-12-31 (AL1)"; resolve the TODO.
- **SAA-C03 relevance:** LOW–MEDIUM
- **Further action:** None

### FOUND-006 — AL2023 description inaccuracies
- **Category:** ERROR (SELinux/cron) / AMBIGUOUS (lineage)
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Amazon Linux → AL2023 bullets; intro "based off CentOS and Fedora"
- **Original claim:** "Security Enhanced Linux (SELinux) enabled by default"; "Source CentOS9 stream"; "Cronie not installed by default"
- **Issue:** SELinux is enabled but in **permissive** mode by default (logs, doesn't enforce). AL2023 is sourced from multiple Fedora versions and other distros including CentOS 9 Stream (not solely CentOS 9). Cron replaced by systemd timers (the cron note is correct but lacks that context).
- **Official evidence:** "By default, SELinux for AL2023 is enabled and set to permissive mode." / "AL2023 is RPM-based and includes components sourced from multiple versions of Fedora and other distributions, such as CentOS 9 Stream." / "`systemd` timers replace `cron`".
- **Source:** SELinux – AL2023 (https://docs.aws.amazon.com/linux/al2023/ug/selinux.html); Comparing AL2 and AL2023 (https://docs.aws.amazon.com/linux/al2023/ug/compare-with-al2.html)
- **Assessment:** gp3 default and Corretto also confirmed on the same comparison page.
- **Suggested correction:** "SELinux enabled (permissive mode by default)"; "Components sourced from multiple Fedora versions and others (e.g. CentOS 9 Stream)".
- **SAA-C03 relevance:** LOW
- **Further action:** None

### FOUND-007 — Reserved Instance discount percentages
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** MEDIUM
- **Location:** EC2 Pricing Models → Reserved / Standard / Convertible
- **Original claim:** "Reserved - best long term (up to 75% off)"; "Standard - up to 75% off"; "Convertible - up to 54% off"
- **Issue:** AWS currently quotes up to 72% (RIs/Standard) and up to 66% (Convertible).
- **Official evidence:** RI page: "significant discount (up to 72%) compared to On-Demand"; RI pricing page: "Convertible Reserved Instances provide… a significant discount (up to 66%)".
- **Source:** https://aws.amazon.com/ec2/pricing/reserved-instances/ ; https://aws.amazon.com/ec2/pricing/reserved-instances/pricing/
- **Assessment:** Marketing figures that change over time; exact numbers are rarely exam-relevant.
- **Suggested correction:** Update to 72% / 66%, or say "significantly less than On-Demand; Standard > Convertible".
- **SAA-C03 relevance:** LOW
- **Further action:** None

### FOUND-008 — EC2-Classic / Scheduled RI references
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** MEDIUM
- **Location:** Reserved → Standard → "Change network EC2-classic -> VPC and vice versa"; "~~Scheduling~~"
- **Original claim:** as quoted
- **Issue:** EC2-Classic was retired (expected complete 15 Aug 2022); the network-platform modification is obsolete. Current modification doc lists AZ, scope, and instance size changes (size only within same family/generation and with platform restrictions). Scheduled RIs are already struck through, fine.
- **Official evidence:** AWS blog: "On August 15, 2022 we expect all migrations to be complete, with no remaining EC2-Classic resources". Modification page lists "Change Availability Zones", "Change the scope", instance size "within the same instance family and generation".
- **Source:** https://aws.amazon.com/blogs/aws/ec2-classic-is-retiring-heres-how-to-prepare/ ; https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ri-modifying.html
- **Assessment:** Blog is a "we expect" statement; the current modify page no longer lists a network-platform change in the excerpt inspected.
- **Suggested correction:** Delete the EC2-Classic bullet; qualify size change "within same family (Linux, regional scope)".
- **SAA-C03 relevance:** LOW
- **Further action:** None

### FOUND-009 — RI Marketplace selling limit
- **Category:** ERROR
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** RI Marketplace
- **Original claim:** "Sell up to $20k per year"
- **Issue:** Limit is $50,000 over the account's lifetime, not an annual limit.
- **Official evidence:** "The following limits… apply to the lifetime of your AWS account. They are not annual limits and they can't be increased. You can sell up to $50,000 in Reserved Instances." Also: only Standard RIs; AWS India customers can't sell. Other bullets (30 days active, US bank) are confirmed.
- **Source:** Reserved Instance Marketplace – EC2 User Guide, https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ri-market-general.html
- **Assessment:** Directly contradicted. GovCloud exclusion was not verified.
- **Suggested correction:** "Sell up to $50,000 (lifetime)".
- **SAA-C03 relevance:** LOW
- **Further action:** None

### FOUND-010 — Hibernation not covered
- **Category:** INCOMPLETE
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Instance Lifecycle (Actions/States)
- **Original claim:** Actions list has no Hibernate; states list has no hibernated/stopped-by-hibernation.
- **Issue:** Hibernation is named explicitly in the exam guide (Task 3.2/4.x "Scaling strategies (for example, auto scaling, hibernation)").
- **Official evidence:** "Hibernation saves the contents from the instance memory (RAM) to your Amazon EBS root volume… When you start your instance: the EBS root volume is restored… the RAM contents are reloaded." Exam guide v1.1 lists hibernation as a knowledge item.
- **Source:** Hibernate your Amazon EC2 instance, https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Hibernate.html ; SAA-C03 Exam Guide v1.1 (URL above)
- **Assessment:** Material omission relative to exam scope.
- **Suggested correction:** Add a "Hibernate" action: saves RAM to encrypted EBS root volume; fast resume; needs supported instance/AMI and RAM size limits (verify details before writing).
- **SAA-C03 relevance:** HIGH
- **Further action:** Verify further (prerequisites/limits)

### FOUND-011 — Fargate "awslogs networking mode"
- **Category:** ERROR
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Fargate → Details
- **Original claim:** "Must use awslogs networking mode"
- **Issue:** `awslogs` is a log driver; the Fargate network mode is `awsvpc` (as the note itself says in ECS task definitions and "ENI per task").
- **Official evidence:** ECS Fargate docs: tasks using `awsvpc` network mode are associated with an elastic network interface, so ELB target groups must use `ip` target type.
- **Source:** Amazon ECS on AWS Fargate, https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html
- **Assessment:** Confirms awsvpc. The statement that Fargate can use only awsvpc is not explicitly quoted from the page I inspected, but the note's other lines are consistent. Interpretation: "awslogs" was a mix-up.
- **Suggested correction:** "Must use awsvpc network mode (log driver awslogs is a separate setting)".
- **SAA-C03 relevance:** MEDIUM
- **Further action:** None

### FOUND-012 — Service Connect "evolution of App Mesh"
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** ECS Service Connect intro
- **Original claim:** "Evolution of App Mesh, abstracting much of the configuration between App Mesh, Cloud Map and ELB."
- **Issue:** App Mesh support ends 2026-09-30 (already past); AWS directs users to Service Connect. The exam guide lists App Mesh under out-of-scope services.
- **Official evidence:** "End of support notice: On September 30, 2026, AWS will discontinue support for AWS App Mesh… you will no longer be able to access the AWS App Mesh console or resources… Migrating from AWS App Mesh to Amazon ECS Service Connect". Exam guide: AWS App Mesh appears after the "Out-of-scope AWS services and features" heading.
- **Source:** What Is AWS App Mesh?, https://docs.aws.amazon.com/app-mesh/latest/userguide/what-is-app-mesh.html ; SAA-C03 Exam Guide v1.1
- **Assessment:** Describing Service Connect as the successor remains reasonable; App Mesh is now discontinued.
- **Suggested correction:** "Replacement for App Mesh (discontinued 30 Sep 2026)…" and keep the rest.
- **SAA-C03 relevance:** MEDIUM
- **Further action:** None

### FOUND-013 — Compute Optimizer supported resources incomplete
- **Category:** INCOMPLETE
- **Verification status:** CONFIRMED
- **Confidence:** MEDIUM
- **Location:** AWS Compute Optimiser
- **Original claim:** "over the last 14 days"; resource list (EC2, ASG, EBS, Lambda, ECS on Fargate, SQL Server licenses)
- **Issue:** 14 days is the default lookback (up to 93 days with the paid enhanced infrastructure metrics); RDS/Aurora and idle-resource identification are now supported.
- **Official evidence:** Supported resources include "Amazon Aurora and Amazon RDS databases"; "…last 14 days… extends lookback to 93 days (compared to the 14-day default)"; "identify idle resources".
- **Source:** What is AWS Compute Optimizer?, https://docs.aws.amazon.com/compute-optimizer/latest/ug/what-is-compute-optimizer.html
- **Assessment:** Existing statements are correct but not complete. Compute Optimizer is in the exam guide's in-scope list.
- **Suggested correction:** Add "default lookback (up to 93 days with enhanced metrics)" and "RDS databases, idle resources".
- **SAA-C03 relevance:** LOW–MEDIUM
- **Further action:** None

### FOUND-014 — ENA speed claim
- **Category:** OUTDATED (candidate)
- **Verification status:** UNVERIFIED
- **Confidence:** LOW
- **Location:** AMI → "ENA… can support up to 100Gb/s"
- **Issue:** Newer instance types exceed 100 Gbps networking; the figure may be stale. Not checked against an official source (ENA doc fetch returned no match).
- **Source:** None inspected
- **Suggested correction:** Verify against the Enhanced networking (ENA) page.
- **SAA-C03 relevance:** LOW
- **Further action:** Verify further

### FOUND-015 — Dedicated Hosts vs Dedicated Instances
- **Category:** INCOMPLETE (candidate)
- **Verification status:** UNVERIFIED
- **Confidence:** LOW
- **Location:** Pricing Models → Dedicated
- **Issue:** Notes don't distinguish Dedicated Instances from Dedicated Hosts (per-socket/core licensing, host affinity). "Can be on-demand, reserved or spot" applies to Dedicated Instances; Hosts have their own purchase options. Not checked.
- **Source:** None inspected
- **SAA-C03 relevance:** MEDIUM
- **Further action:** Verify further

### FOUND-016 — Elastic Beanstalk "Not recommended for Production"
- **Category:** AMBIGUOUS (candidate)
- **Verification status:** UNVERIFIED
- **Confidence:** LOW
- **Location:** Elastic Beanstalk intro
- **Issue:** Unqualified statement; AWS documents production use (e.g. load-balanced environments). Not checked.
- **Source:** None inspected
- **SAA-C03 relevance:** LOW (Beanstalk is in exam guide in-scope list)
- **Further action:** Verify further
