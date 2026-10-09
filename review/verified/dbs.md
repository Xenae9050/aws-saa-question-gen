# Verification Report: Databases

- **Notes section:** `notes/working/dbs.md`
- **Triage report:** `review/findings/dbs.md`
- **Verification date:** 2026-10-09
- **Exam scope authority:** AWS Certified Solutions Architect – Associate (SAA-C03) exam guide on docs.aws.amazon.com (no version/date shown on the pages fetched)

## 1. Summary table

| Finding ID | Verification outcome | Recommended disposition | Priority | Change since triage? |
| ---------- | -------------------- | ----------------------- | -------- | -------------------- |
| FOUND-001  | CONFIRMED            | CORRECT                 | HIGH     | NO                   |
| FOUND-002  | CONFIRMED            | CORRECT                 | HIGH     | YES                  |
| FOUND-003  | CONFIRMED            | CORRECT                 | HIGH     | YES                  |
| FOUND-005  | CONFIRMED            | CORRECT                 | HIGH     | NO                   |
| FOUND-006  | CONFIRMED            | CORRECT                 | MEDIUM   | YES                  |
| FOUND-007  | CONFIRMED            | CORRECT                 | MEDIUM   | NO                   |
| FOUND-008  | CONFIRMED            | CORRECT                 | MEDIUM   | NO                   |
| FOUND-009  | PARTIALLY SUPPORTED  | CORRECT                 | LOW      | NO                   |
| FOUND-010  | CONFIRMED            | CLARIFY                 | MEDIUM   | NO                   |
| FOUND-011  | CONFIRMED            | REMOVE_OR_DEPRIORITISE  | MEDIUM   | YES                  |
| FOUND-012  | CONFIRMED            | EXPAND                  | HIGH     | NO                   |
| FOUND-013  | PARTIALLY SUPPORTED  | EXPAND                  | HIGH     | YES                  |
| FOUND-014  | PARTIALLY SUPPORTED  | CLARIFY                 | MEDIUM   | YES                  |
| FOUND-015  | PARTIALLY SUPPORTED  | REMOVE_OR_DEPRIORITISE  | LOW      | YES                  |

(FOUND-004 was intentionally unused in triage.) No triage finding was rejected, so there are no false-positive entries.

## 2. Details for changed findings

### FOUND-002 — "Encryption in transit provided by default"

**Change from triage:** Per-engine enforcement defaults, which triage left unchecked, are now evidenced. The correction needs to say enforcement is engine-specific, not just client-initiated.
**Original assessment:** CONFIRMED, MEDIUM. TLS is client-initiated; per-engine defaults unchecked.
**Verified conclusion:** Technically wrong as a blanket statement. TLS is supported on all engines, and whether it is enforced by default varies by engine and version.
**Official evidence:** The RDS TLS page describes TLS as something you use "from your application", with engine-specific setup. RDS for PostgreSQL has `rds.force_ssl` defaulting to 1 (on) for version 15 and later, and 0 (off) for 14 and older. RDS for SQL Server has `rds.force_ssl` defaulting to 0 (off).
**Source:**

- Using SSL/TLS to encrypt a connection to a DB instance or cluster – https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.SSL.html
- Using SSL with a PostgreSQL DB instance – https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/PostgreSQL.Concepts.General.SSL.html ("The rds.force_ssl parameter default value is 1 (on) for RDS for PostgreSQL version 15 and later")
- Using SSL with a Microsoft SQL Server DB instance – https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/SQLServer.Concepts.General.SSL.Using.html ("By default, the rds.force_ssl parameter is set to 0 (off)")

**Recommended disposition:** CORRECT
**Proposed replacement:** "Encryption in transit – SSL/TLS is supported on all engines; clients must connect using TLS. Enforcement is engine-specific and can be configured via parameter groups (e.g. `rds.force_ssl`)."
**Remaining uncertainty:** MySQL and MariaDB enforcement defaults (`require_secure_transport`) were not stated on the engine pages inspected. The replacement deliberately avoids claiming them.

### FOUND-003 — Aurora Serverless v2 "does not scale to 0"

**Change from triage:** The 0 ACU minimum depends on engine version. Triage's correction ("range 0–256") would be wrong for older versions.
**Original assessment:** CONFIRMED, HIGH. Capacity is 0–256 ACUs and the cluster can scale to 0.
**Verified conclusion:** Both statements in the notes (min 0.5 ACU, 0.5–128 range) are outdated as general statements. They were accurate for older engine versions.
**Official evidence:** "Aurora serverless offers capacity from 0 ACUs to 256 ACUs. With the minimum capacity of 0 ACUs, the cluster will scale to 0 when there is no workload running." The version table lists 0.5–128 for Aurora MySQL 3.02.0+, 0.5–256 for 3.06.0+, and 0–256 for 3.08.0+ (Aurora PostgreSQL 13.15+, 14.12+, 15.7+, 16.3+). The auto-pause page says instances scale to zero ACUs and pause after a period with no user connections, with no charge for instance capacity while paused. Resume happens when a connection request arrives, so brief connection delays are possible. The ~2 GiB-per-ACU claim in the notes is correct.
**Source:**

- How Aurora serverless works – https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.how-it-works.html (Capacity section)
- Scaling to Zero ACUs with automatic pause and resume – https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2-auto-pause.html

**Recommended disposition:** CORRECT
**Proposed replacement:** "Can scale down to 0 ACUs and automatically pause when idle if minimum capacity is set to 0 (newer engine versions; otherwise minimum is 0.5 ACUs). Capacity range up to 256 ACUs." Table row: "0–256 ACUs (version dependent)".
**Remaining uncertainty:** None material. The exact resume latency was not stated on the pages inspected.

### FOUND-006 — Redshift node types and sizing

**Change from triage:** The node-limit check triage left open is resolved, and it shows the note's sizing figures are internally inconsistent. DS2 is explicitly unavailable.
**Original assessment:** CONFIRMED, HIGH. dc/ds description is stale; node limits unchecked.
**Verified conclusion:** Technically outdated (dense storage) and partly inaccurate (sizing). `dc2.large` is 160 GB per node with a range of 1–32 nodes, so "single node – 160GB" and "up to 32" are accurate for that size only. "Up to 128 (each 160GB)" is not: the 2–128 range belongs to `dc2.8xlarge` (2.56 TB per node). Current guidance recommends RG/RA3 (managed storage) and DC2 only for datasets under 1 TB.
**Official evidence:** "we recommend choosing RG or RA3… scaling and paying for compute and managed storage independently"; "For datasets under 1 TB (compressed), we recommend DC2 node types"; "Dense storage (DS2) node types are no longer available."
**Source:** Amazon Redshift provisioned clusters – https://docs.aws.amazon.com/redshift/latest/mgmt/working-with-clusters.html (node type specification tables and DS2 note)
**Recommended disposition:** CORRECT
**Proposed replacement:** Replace the dc/ds bullets with: "RA3 / RG – managed storage (local SSD plus S3), compute and storage scale independently (recommended). DC2 – local SSD, best for datasets under 1 TB. (DS2 no longer available.)" Reword the 160 GB / 32 / 128 node bullets to avoid implying one node size.
**Remaining uncertainty:** The Redshift Serverless one-liner suggested by triage was not verified. Serverless appears only as a passing reference on the pages inspected, and no Serverless feature page was read. No replacement text is proposed for it. Per-account default node quotas were not checked.

### FOUND-011 — QLDB presented as a current service

**Change from triage:** UNVERIFIED, from memory → CONFIRMED against an official AWS page.
**Original assessment:** Candidate end of support (believed 31 Jul 2025); not live-verified. Confidence MEDIUM.
**Verified conclusion:** Technically outdated. QLDB is listed as a service in full shutdown, with end-of-support date July 31, 2025. The AWS definition of "full shutdown" is "completely removed from the AWS portfolio and are no longer available or supported in any capacity." QLDB documentation URLs I tried (`/qldb/latest/developerguide/...`) returned 404, which is consistent with this. QLDB is not in the SAA-C03 in-scope services list.
**Official evidence:** "Services in Full Shutdown" table row: "Amazon Quantum Ledger Database (Amazon QLDB) – July 31, 2025".
**Source:** Services in Full Shutdown – AWS General Reference – https://docs.aws.amazon.com/general/latest/gr/full_shutdown_services.html
**Recommended disposition:** REMOVE_OR_DEPRIORITISE. The author should either delete the section or keep a one-line note that it is discontinued. It has no SAA value.
**Proposed replacement:** If retained: "Amazon QLDB – reached end of support on 31 July 2025 and is no longer available."
**Remaining uncertainty:** The AWS-recommended migration target (triage mentioned Aurora PostgreSQL) was not verified, so none is proposed.

### FOUND-013 — DynamoDB capacity modes, DAX, global tables, partition numbers

**Change from triage:** Part of the "unverified partition numbers" allegation is rejected. The per-partition throughput maximum is documented, so the note's 1000 WCU figure is correct.
**Original assessment:** CONFIRMED (capacity-mode omission) / UNVERIFIED (other items), MEDIUM.
**Verified conclusion:**

- Capacity modes: technically correct but incomplete. The notes never mention on-demand versus provisioned.
- "1000 WCUs per partition": correct and should remain.
- The "10GB" per-partition split trigger: not evidenced on any page inspected. It is unresolved and should be neither confirmed nor removed.
- DAX and Global Tables not being named: not verified.

**Official evidence:** "On-demand mode is the default and recommended throughput option for most DynamoDB workloads." In provisioned mode you specify reads and writes per second. "Every partition in a DynamoDB table is designed to deliver a maximum capacity of 3,000 read units per second and 1,000 write units per second." The partitions page says additional partitions are allocated when throughput is raised beyond what existing partitions support, or when a partition fills to capacity, and gives no size figure.
**Source:**

- DynamoDB throughput capacity – https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/capacity-mode.html
- Partition key design – https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-partition-key-design.html
- Partitions and data distribution – https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.Partitions.html

**Recommended disposition:** EXPAND
**Proposed replacement:** Add under Features: "Capacity modes – on-demand (pay per request, no capacity planning, unpredictable workloads) or provisioned (specify RCU/WCU, predictable workloads)". Leave the 1000 WCU statement unchanged. The note's RCU maximum is not stated numerically, so adding "3,000 RCUs" is optional.
**Remaining uncertainty:** The 10 GB split trigger and the DAX / Global Tables omissions.

### FOUND-014 — Strongly consistent reads / Query description

**Change from triage:** UNRESOLVED → PARTIALLY SUPPORTED. The consistency documentation (not inspected at triage) has now been read.
**Original assessment:** AMBIGUOUS, LOW confidence; evidence insufficient.
**Verified conclusion:**

- Ambiguous but not wrong. "Wait until copies are consistent" is a loose mental model. The docs describe a strongly consistent read as returning "the most up-to-date data, reflecting the updates from all prior write operations that were successful."
- Omission: strongly consistent reads are supported only on tables and local secondary indexes. They are not supported on global secondary indexes or streams. This is a useful SAA point.
- Query "with a composite primary key": not established as an error. The Query docs require a partition key value and optionally apply a sort-key condition. They do not state that simple-primary-key tables cannot be queried. The triage claim that they can is therefore unsupported by the pages inspected, and no change is proposed to that line.

**Official evidence:** "Strongly consistent reads are only supported on tables and local secondary indexes. Strongly consistent reads from a global secondary index or a DynamoDB stream are not supported." Eventually consistent reads cost half of strongly consistent reads.
**Source:**

- DynamoDB read consistency – https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadConsistency.html
- Querying tables in DynamoDB – https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Query.html

**Recommended disposition:** CLARIFY
**Proposed replacement:** "Strongly consistent – returns the most up-to-date data reflecting all prior successful writes. Slower; not supported on global secondary indexes." Leave the Query bullet unchanged.
**Remaining uncertainty:** The "all copies consistent within a second" wording is not addressed by the pages inspected.

### FOUND-015 — Content beyond SAA depth / scope

**Change from triage:** UNRESOLVED → PARTIALLY SUPPORTED. The factual scope-list claims are confirmed. The depth judgement remains interpretive, and the exam-guide task statements were read.
**Original assessment:** UNRESOLVED, MEDIUM. Scope inference is interpretive.
**Verified conclusion:**

- The in-scope Database list contains Aurora, Aurora Serverless, DocumentDB, DynamoDB, ElastiCache, Keyspaces, Neptune, RDS and Redshift. QLDB and MemoryDB are not named.
- The Analytics list includes Athena, Data Exchange, Glue and Lake Formation.
- AWS CodeCommit appears on the out-of-scope list, so the Glue Studio "AWS Code Commit" bullet (notes line 217) is low value.
- The lists are non-exhaustive, so MemoryDB should not be dropped on that basis alone. Task 3.3 covers "caching strategies", "in-memory" and "serverless" database types, so an overview of MemoryDB is semantically relevant.
- QLDB is moot because it has been shut down (FOUND-011).
- That MongoDB, Cassandra or Gremlin internals, Babelfish and Glue DPU details exceed SAA depth is a reasonable judgement but is not stated by AWS.

**Official evidence:** In-scope and out-of-scope service lists; exam guide Domain 3, Task 3.3 knowledge statements ("Caching strategies and services (for example, Amazon ElastiCache)", "Database capacity planning", "Database connections and proxies", "Database replication (for example, read replicas)", "Database types and services (for example, serverless, relational compared with non-relational, in-memory)").
**Source:**

- In-Scope AWS Services – https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/saa-03-in-scope-services.html
- Out-of-Scope AWS Services – https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/saa-03-out-of-scope-services.html
- Content Domain 3 – https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/solutions-architect-associate-03-domain3.html

**Recommended disposition:** REMOVE_OR_DEPRIORITISE (the CodeCommit bullet and the QLDB section only; deprioritise the deep-internals content for study, do not delete it).
**Proposed replacement:** No text replacement proposed.

## 3. Verification notes on unchanged findings

These findings are unchanged from triage. They are listed here only to say what evidence was re-inspected, and no full entries are given.

- **FOUND-001:** Re-fetched Concepts.MultiAZ and multi-az-db-clusters-concepts. Multi-AZ DB _cluster_ is an RDS option (writer plus two readable standbys in three AZs) and "aren't the same as Aurora DB clusters".
- **FOUND-005:** Re-fetched the Multi-AZ Redshift page. RG/RA3 provisioned clusters support Multi-AZ via a single endpoint, with a 99.99% SLA versus 99.9% for Single-AZ.
- **FOUND-007:** The MySQL and MariaDB read replica pages say up to 15 replicas. The Quotas page says "Read replicas per primary: 15" (Aurora not adjustable; Oracle 15 but 5 recommended). The PostgreSQL engine page gave no specific number, so it is covered only by the generic quota.
- **FOUND-008:** The Aurora Global Database page says "up to 10 secondary DB clusters in different Regions", with latency typically under a second.
- **FOUND-009:** The Quotas page confirms the 1,000 requests/s and 500 concurrent limits apply only to Aurora Serverless v1. The Data API pages for current cluster types state no rate limit, but also do not say "unlimited". The note's claim is therefore unsupported. They also confirm credentials come from Secrets Manager and that no persistent connection is needed. Triage's suggested optional addition is supported.
- **FOUND-010:** The Aurora serverless page says a cluster with no serverless instances is "provisioned… the oldest and most common kind of DB cluster". It does not say "default", and it describes mixed-configuration clusters.
- **FOUND-012:** All four sub-claims were confirmed. ElastiCache supports Valkey, Memcached and Redis OSS. Serverless caches can use public endpoints (IAM auth and TLS 1.3 required, Valkey 9.0+). Node-based Valkey supports durability via a Multi-AZ transactional log. MemoryDB offers "microsecond read and single-digit millisecond write latency" and is compatible with Valkey and Redis OSS. The ElastiCache write latency implied by "microseconds" in the notes was not inspected.

## 4. Totals and limitations

- **Findings reviewed:** 14
- **CONFIRMED:** 10 (001, 002, 003, 005, 006, 007, 008, 010, 011, 012)
- **PARTIALLY SUPPORTED:** 4 (009, 013, 014, 015)
- **REJECTED:** 0
- **UNRESOLVED:** 0
- **Materially changed since triage:** 7 (002, 003, 006, 011, 013, 014, 015)

**Live-verification limitations**

- All cited pages were fetched live on 2026-10-09. AWS docs pages show no publication dates, and the exam guide pages show no version number, so a newer guide version could not be ruled out.
- The QLDB developer-guide URLs return 404, and a QLDB blog URL redirected to a generic page. QLDB's shutdown is evidenced only by the General Reference "Services in Full Shutdown" page.
- MySQL and MariaDB TLS enforcement defaults were not stated on the engine pages inspected.
- Not verified:
  - DynamoDB's 10 GB partition trigger.
  - The DAX and Global Tables omissions.
  - Redshift per-account node quotas.
  - Redshift Serverless features.
  - ElastiCache write-latency comparison in the MemoryDB note.
- Claims in the notes outside the triage findings (Athena, Lake Formation, MongoDB, Neptune, most Glue and DMS content) were not verified and are not endorsed.
- Triage's "items noted but not researched" (Aurora vCPU/memory figures, RDS storage maximums, the replication-feature table, the Secrets Manager exclusions) were out of scope for this verification and remain open.
- Images in the notes were not assessed.
