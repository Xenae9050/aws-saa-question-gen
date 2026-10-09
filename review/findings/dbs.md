# Triage: Databases (notes/working/dbs.md)

## Summary

- Section reviewed: `notes/working/dbs.md` (QLDB, ElastiCache, MemoryDB, Redshift, Athena, Data Exchange, Glue, Lake Formation, RDS, Aurora, DocumentDB, MongoDB, DynamoDB, Keyspaces, Neptune, DMS)
- Official exam guide checked: AWS Certified Solutions Architect – Associate (SAA-C03) exam guide and in/out-of-scope service lists, https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/solutions-architect-associate-03.html (no version date shown on the pages fetched)
- Live verification status: Partial. Web access worked; 19 official AWS pages were opened. Several pages 404'd or redirected (QLDB docs, DynamoDB docs redirect), so some candidates remain UNVERIFIED.
- Findings: 14 total (FOUND-004 intentionally unused)
  - ERROR: 1 CONFIRMED HIGH (001), 1 CONFIRMED MEDIUM (002)
  - OUTDATED: 8 CONFIRMED (003, 005–010, 012; 009 and 013 partly), 1 UNVERIFIED (011)
  - INCOMPLETE: 1 partly CONFIRMED (013)
  - AMBIGUOUS: 1 UNRESOLVED (014)
  - EXAM SCOPE: 1 UNRESOLVED (015)
  - Categories shown on some entries as "X / Y" reflect the primary and secondary classification
- Important limitations:
  - Only claims judged likely wrong/stale were researched; the rest of the section (Athena, Lake Formation, MongoDB, Neptune, most Glue/DMS detail) was not individually verified and is not asserted correct.
  - Exam-guide pages inspected were the scope lists; task-statement pages (domain 1–4) were not read in full.
  - The section is a mix of SAA-relevant services and detail well beyond SAA depth (Babelfish, MongoDB internals, Cassandra internals, Neptune query languages). See FOUND-015.

## Findings

### FOUND-001 — Multi-AZ "cluster" deployment attributed to Aurora only
- **Category:** ERROR
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** RDS → Multi AZ
- **Original claim:** "Multi AZ instances deployment is for RDS Instances / Multi AZ cluster deployment is for Aurora Clusters"
- **Issue:** "Multi-AZ DB cluster" is an RDS deployment option (one writer + two readable standbys in three AZs), not an Aurora feature. Aurora is a separate architecture. The notes' Multi-AZ vs Read Replica table also says standby "Always span 2 AZs" and is not readable, which is only true of Multi-AZ DB *instance* deployments.
- **Official evidence:** "A Multi-AZ DB cluster deployment has standby DB instances that provide failover support and can also serve read traffic." "Multi-AZ DB clusters aren't the same as Aurora DB clusters." Uses semisynchronous replication, three AZs.
- **Source:** Configuring and managing a Multi-AZ deployment for Amazon RDS – https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html ; Multi-AZ DB cluster deployments for Amazon RDS – https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/multi-az-db-clusters-concepts.html
- **Assessment:** Directly contradicts the note.
- **Suggested correction:** "Multi-AZ DB instance deployment: one non-readable standby (RDS). Multi-AZ DB cluster deployment: two readable standbys in 3 AZs (RDS, supported engines only). Aurora clusters are different: shared storage across 3 AZs." Qualify the comparison table as "Multi-AZ DB instance".
- **SAA-C03 relevance:** HIGH – Multi-AZ vs read replica vs Aurora is a core resilience topic.
- **Further action:** None

### FOUND-002 — "Encryption in transit provided by default"
- **Category:** ERROR
- **Verification status:** CONFIRMED
- **Confidence:** MEDIUM
- **Location:** RDS → Encryption
- **Original claim:** "Encryption in transit – Provided by default via databases DNS endpoint"
- **Issue:** SSL/TLS is supported on all engines but the client must use it (and optionally verify the cert bundle); it is not automatically applied to every connection by virtue of the DNS endpoint.
- **Official evidence:** "You can use SSL or TLS from your application to encrypt a connection to a database…" and steps require choosing a CA, downloading a bundle and connecting using the engine's SSL process.
- **Source:** Using SSL/TLS to encrypt a connection to a DB instance or cluster – https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.SSL.html
- **Assessment:** Page describes TLS as client-initiated; whether specific engines enforce it by default (e.g. require_secure_transport/rds.force_ssl) varies by engine and version and was not checked.
- **Suggested correction:** "Encryption in transit: supported via SSL/TLS; clients must connect using TLS (can be enforced via parameter groups)." Verify the enforcement parameter wording per engine before adding.
- **SAA-C03 relevance:** HIGH – security domain.
- **Further action:** Verify further (per-engine enforcement defaults)

### FOUND-003 — Aurora Serverless v2 "does not scale to 0"
- **Category:** ERROR (now OUTDATED statement; stated as fact in two places)
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Aurora → Aurora Serverless v2; Serverless vs. Provisioned table ("0.5-128 ACUs")
- **Original claim:** "Does not scale to 0 - must maintain at least 0.5 ACUs"; "Capacity range 0.5-128 ACUs"
- **Issue:** Current docs state capacity from 0 to 256 ACUs, and that with minimum 0 ACUs the cluster scales to 0 when idle.
- **Official evidence:** "Aurora serverless offers capacity from 0 ACUs to 256 ACUs. With the minimum capacity of 0 ACUs, the cluster will scale to 0 when there is no workload running." Each ACU ≈ 2 GiB memory (note's 2 GiB is correct).
- **Source:** How Aurora serverless works – https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.how-it-works.html
- **Assessment:** Direct conflict.
- **Suggested correction:** "Can scale to 0 ACUs (auto-pause) when min capacity is set to 0; range 0–256 ACUs." Cost/resume latency details not verified.
- **SAA-C03 relevance:** HIGH – serverless/cost optimisation; Aurora Serverless is in the exam scope list.
- **Further action:** None

### FOUND-005 — Redshift is "single-AZ"
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Amazon Redshift – "Redshift is **single-AZ.** Would need to run clone in different AZ manually."
- **Issue:** Redshift supports Multi-AZ deployments for provisioned RA3 (and newer RG) clusters, accessed via a single endpoint.
- **Official evidence:** "Amazon Redshift supports multiple Availability Zones (Multi-AZ) deployments for provisioned RG or RA3 clusters… compute resources in two AZs… single endpoint… SLA 99.99% vs 99.9% single-AZ."
- **Source:** Multi-AZ deployment – https://docs.aws.amazon.com/redshift/latest/mgmt/managing-cluster-multi-az.html
- **Assessment:** Direct conflict. Single-AZ is still the case for other node types/configs (not verified which).
- **Suggested correction:** "Single-AZ by default; Multi-AZ available for RA3/RG provisioned clusters."
- **SAA-C03 relevance:** HIGH – resilience.
- **Further action:** None

### FOUND-006 — Redshift node types and sizing
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Redshift → Configurations / Node types
- **Original claim:** "Dense compute (dc) / Dense storage (ds)… Single node – 160GB … Up to 32 by default… (each 160GB)"
- **Issue:** Current guidance recommends RA3/RG (managed storage, compute and storage scaled independently) with DC2 for small (<1 TB) datasets; DS node types are not mentioned. Per-node 160 GB figures relate to old dc1/dc2.large nodes. Redshift Serverless is not mentioned.
- **Official evidence:** "we recommend choosing RG or RA3… RG and RA3 nodes with managed storage enable you to… scale and pay for compute and managed storage independently… DC2… for datasets under 1 TB."
- **Source:** Amazon Redshift provisioned clusters – https://docs.aws.amazon.com/redshift/latest/mgmt/working-with-clusters.html
- **Assessment:** Node-type description is stale. Node count limits were not checked (UNVERIFIED part).
- **Suggested correction:** Replace dc/ds bullets with RA3 (managed storage, S3-backed) / DC2 (small data). Add one line: Redshift Serverless (not in notes).
- **SAA-C03 relevance:** MEDIUM – Redshift in scope; node-level detail low, serverless vs provisioned medium.
- **Further action:** Verify further (node limits; Serverless one-liner)

### FOUND-007 — Read replica limit "5 for MySQL, MariaDB & PostgreSQL"
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** RDS → Read Replicas
- **Original claim:** "Up to 5 replicas for MySQL, MariaDB & PostgreSQL dbs"
- **Issue:** RDS for MySQL allows up to 15 read replicas per source in a Region; the Quotas page lists "Read replicas per primary: 15". Oracle also 15 (5 recommended). MariaDB/PostgreSQL not individually inspected.
- **Official evidence:** MySQL page: "You can create up to 15 read replicas from one DB instance within the same Region." Quotas page: 15 per primary; Aurora quota not adjustable (note's 15 for Aurora is consistent).
- **Source:** Working with MySQL read replicas – https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_MySQL.Replication.ReadReplicas.html ; Quotas and constraints for Amazon RDS – https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_Limits.html
- **Assessment:** The 5 limit is stale for at least MySQL.
- **Suggested correction:** "Up to 15 replicas per source (RDS) and up to 15 for Aurora."
- **SAA-C03 relevance:** MEDIUM
- **Further action:** None

### FOUND-008 — Aurora Global Database "up to 5 secondary clusters"
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Aurora → Global Database
- **Original claim:** "Has up to 5 secondary db clusters in different regions"
- **Issue:** Current docs state up to 10 secondary Regions.
- **Official evidence:** "…a primary DB cluster in one Region, and up to 10 secondary DB clusters in different Regions… latency typically under a second." Note's replication-latency statement is consistent.
- **Source:** Using Amazon Aurora Global Database – https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html
- **Assessment:** Direct conflict. Worth also mentioning switchover/failover and write forwarding (not in notes) – optional.
- **Suggested correction:** Change 5 → 10.
- **SAA-C03 relevance:** MEDIUM
- **Further action:** None

### FOUND-009 — RDS Data API described as "Unlimited requests per second"
- **Category:** OUTDATED / ERROR
- **Verification status:** CONFIRMED (partly)
- **Confidence:** MEDIUM
- **Location:** Aurora → RDS Data API
- **Original claim:** "Unlimited requests per second"
- **Issue:** The RDS Quotas page lists Data API quotas (1,000 requests/s, 500 concurrent), but states these apply only to Aurora Serverless **v1**. Whether v2/provisioned clusters are unlimited is not stated on the pages inspected, so "unlimited" is unsupported.
- **Official evidence:** Quotas table "Data API requests per second: 1,000 … only applies to Aurora Serverless v1 clusters." Data API overview describes HTTP endpoint, no persistent connection, credentials from Secrets Manager.
- **Source:** Quotas and constraints for Amazon RDS – https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_Limits.html ; Using the Amazon RDS Data API – https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/data-api.html
- **Assessment:** Claim not supported; safest to remove the "unlimited" claim. Notes also omit that credentials come from Secrets Manager (useful SAA point).
- **Suggested correction:** Delete "Unlimited requests per second"; optionally add "Uses credentials stored in Secrets Manager; no persistent connection needed".
- **SAA-C03 relevance:** LOW–MEDIUM
- **Further action:** Verify further (v2/provisioned limits)

### FOUND-010 — Aurora "Aurora Serverless Provisioned is the default compute configuration"
- **Category:** AMBIGUOUS → recorded as OUTDATED (terminology)
- **Verification status:** CONFIRMED
- **Confidence:** MEDIUM
- **Location:** Aurora → paragraph beginning "Aurora Serverless Provisioned is the default…"
- **Original claim:** "Aurora Serverless Provisioned is the default compute configuration for Aurora."
- **Issue:** Docs distinguish *provisioned* (oldest/most common) from *serverless* (v2). "Serverless Provisioned" conflates them. Docs also describe mixed-configuration clusters (serverless + provisioned instances), not in notes.
- **Official evidence:** "If you don't use Aurora serverless at all in a DB cluster, all the writers and readers… are provisioned. This is the oldest and most common kind of DB cluster." Serverless and provisioned can be mixed.
- **Source:** How Aurora serverless works – https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.how-it-works.html
- **Assessment:** Wording error; "default" claim itself not explicitly stated.
- **Suggested correction:** "Provisioned is the traditional/default compute configuration for Aurora."
- **SAA-C03 relevance:** MEDIUM
- **Further action:** None

### FOUND-011 — QLDB present as a current service
- **Category:** OUTDATED
- **Verification status:** UNVERIFIED
- **Confidence:** MEDIUM (from memory; NOT live-verified)
- **Location:** "Amazon Quantum Ledger Database" (top of file)
- **Original claim:** Entire section describes QLDB as available
- **Issue:** Candidate: QLDB end-of-support (believed 31 Jul 2025; AWS recommends Aurora PostgreSQL). Attempts to open QLDB developer-guide URLs returned 404 and an AWS blog URL redirected, so this could not be inspected. QLDB is also absent from the SAA-C03 in-scope Database list (list is non-exhaustive) and not on the out-of-scope list.
- **Official evidence:** None inspected for end-of-support. Exam in-scope list (inspected) does not include QLDB.
- **Source:** In-Scope AWS Services – https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/saa-03-in-scope-services.html
- **Assessment:** Do not apply until end-of-support is confirmed from an official AWS page.
- **Suggested correction:** Pending verification; likely add a deprecation note or remove (author's decision). Also fix "cryptographically viable" → "verifiable".
- **SAA-C03 relevance:** LOW (not listed in scope)
- **Further action:** Verify further

### FOUND-012 — ElastiCache engines/features outdated; MemoryDB description
- **Category:** OUTDATED / INCOMPLETE
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** ElastiCache (intro, "Caching types", "Only accessible by resources in same VPC"); MemoryDB
- **Original claim:** "Memcached or Redis"; "Only accessible by resources in same VPC"; MemoryDB "Writes are slower… (milliseconds vs microseconds)"
- **Issue:** (a) ElastiCache now supports Valkey, Memcached and Redis OSS; (b) Serverless caches can use a public endpoint (requires IAM auth + TLS 1.3, Valkey 9.0+), so "only accessible in same VPC" is no longer universally true (VPC remains the default/typical model); (c) node-based Valkey can enable durability via a Multi-AZ transaction log; (d) MemoryDB is compatible with Valkey and Redis OSS, with microsecond reads and single-digit-ms writes — the note's wording compares writes to ElastiCache "microseconds" which is imprecise.
- **Official evidence:** ElastiCache "works with the Valkey, Memcached, and Redis OSS engines"; serverless public endpoints; "microsecond read and single-digit millisecond write latency" for MemoryDB.
- **Source:** What is Amazon ElastiCache? – https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/WhatIs.html ; What is MemoryDB – https://docs.aws.amazon.com/memorydb/latest/devguide/what-is-memorydb.html
- **Assessment:** Direct evidence for (a)–(d). Note: MemoryDB is not in the SAA-C03 in-scope list (non-exhaustive); ElastiCache is.
- **Suggested correction:** Add Valkey as an engine alongside Redis; qualify "VPC only" with "(serverless caches can optionally expose a public endpoint)"; MemoryDB: "microsecond reads, single-digit ms writes".
- **SAA-C03 relevance:** HIGH (ElastiCache caching strategy); MEDIUM (MemoryDB).
- **Further action:** None

### FOUND-013 — DynamoDB: capacity modes, DAX, global tables, partition-split numbers
- **Category:** INCOMPLETE
- **Verification status:** CONFIRMED (omission of capacity modes) / UNVERIFIED (other items)
- **Confidence:** MEDIUM
- **Location:** DynamoDB (Features list, Partitions)
- **Original claim:** "Creates new partitions: For every 10GB… 1000 WCUs per partition"; Features: "Multi-region… In-memory caching"
- **Issue:** (a) No mention of on-demand vs provisioned capacity modes; docs call on-demand "the default and recommended throughput option for most DynamoDB workloads". (b) The partition-splitting numbers are not stated on the partitions page, which says only that partitions are added when throughput or storage requirements exceed existing partitions; treat exact numbers as unverified/internal detail. (c) DAX, global tables, Streams, PITR, TTL are not named (notes only say "multi-region"/"in-memory caching") — not verified here.
- **Official evidence:** Capacity mode page: on-demand = pay-per-request, no capacity planning, default; provisioned = specify RCU/WCU for predictable workloads. Partitions page: additional partitions allocated when throughput increased or partition fills; no numbers.
- **Source:** DynamoDB throughput capacity – https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/capacity-mode.html ; Partitions and data distribution – https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.Partitions.html
- **Assessment:** Capacity-mode omission is the SAA-relevant gap (ties to cost). Partition numbers should not be removed or kept without confirmation.
- **Suggested correction:** Add short "Capacity modes: on-demand (unpredictable, pay per request) vs provisioned (predictable, optional auto scaling)". Name DAX and Global Tables in the Features list only if the author wants.
- **SAA-C03 relevance:** HIGH – DynamoDB is core; cost/performance trade-offs.
- **Further action:** Verify further (DAX, global tables, partition numbers)

### FOUND-014 — Strongly consistent reads / Query description
- **Category:** AMBIGUOUS
- **Verification status:** UNRESOLVED
- **Confidence:** LOW
- **Location:** DynamoDB → Reads; Queries vs. Scans
- **Original claim:** "attempt to read will wait until copies are consistent…"; "Query any table or secondary index with a composite primary key"
- **Issue:** Strongly consistent reads return the most recent write; "wait until copies are consistent" is a loose mental model. Query requires a partition key value (tables with a simple primary key can also be queried with just the partition key – the partitions page says Query reads items sharing a partition key value). The "composite" restriction may mislead. Strong consistency is also unavailable on global secondary indexes (not verified).
- **Official evidence:** Partitions page confirms Query = same partition key value with optional sort-key condition; does not address strong-consistency wording or GSI limits.
- **Source:** https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.Partitions.html
- **Assessment:** Evidence inspected is insufficient for the consistency wording.
- **Suggested correction:** Not proposed until consistency docs are inspected.
- **SAA-C03 relevance:** MEDIUM
- **Further action:** Verify further

### FOUND-015 — Content beyond SAA depth; services not in exam scope list
- **Category:** EXAM SCOPE
- **Verification status:** UNRESOLVED
- **Confidence:** MEDIUM
- **Location:** QLDB, MongoDB internals, Keyspaces (Cassandra internals), Neptune query languages / ML, Babelfish, Glue DPU counts, Athena SerDe; also AWS Glue Studio "AWS Code Commit"
- **Original claim:** e.g. "Version control pipelines using: AWS Code Commit…"
- **Issue:** The in-scope Database list contains Aurora (and Serverless), DocumentDB, DynamoDB, ElastiCache, Keyspaces, Neptune, RDS, Redshift. QLDB and MemoryDB are not listed (list is explicitly non-exhaustive, so this is not proof of exclusion). Analytics list includes Athena, Data Exchange, Glue, Lake Formation. DMS is in scope. AWS CodeCommit is on the **out-of-scope** list; the Glue Studio version-control bullet is low-value. Detailed MongoDB, Cassandra, Gremlin and Babelfish content is likely deeper than SAA expects.
- **Official evidence:** In-scope and out-of-scope lists inspected.
- **Source:** https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/saa-03-in-scope-services.html ; https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/saa-03-out-of-scope-services.html
- **Assessment:** Scope inference is interpretive; no claim of removal. Separate (not inspected): CodeCommit service status, which the CodeCommit page fetched did not address.
- **Suggested correction:** None; consider flagging low-priority content for study-priority only (not deletion).
- **SAA-C03 relevance:** MEDIUM
- **Further action:** Review exam-scope interpretation

## Items noted but not researched (UNVERIFIED candidates, no finding raised)

- Aurora "computing resources scale up to 32 vCPUs and 244GB memory" and "up to 64TB/128TB": RDS quotas page mentions Aurora 128 TiB cluster volume; vCPU claim looks stale but not confirmed.
- RDS storage "max 64TB" and "40 instances" (the 40-instance default was confirmed on the Quotas page, with per-engine/licence limits).
- Replication-feature table (Postgres "Automatic backup: No", "Parallel replication") – likely stale; read-replica pages for each engine not inspected. The notes' own "Replicas must have automatic backups enabled" for MySQL is consistent with the MySQL page.
- RDS Secrets Manager list – note says "does not work with SQL Server"; the current limitations list names read replica creation (except SQL Server/Db2), Blue/Green, RDS Custom and Oracle Data Guard switchover. Verified: 7-day default rotation, secret deleted with DB. The "SQL Server" and "Oracle CDB" exclusions appear stale/unsupported by the page – raise as a further-verification item.
- IAM DB auth: 15-minute token lifetime and MySQL/MariaDB/PostgreSQL support CONFIRMED correct (UsingWithRDS.IAMDBAuth).
- RDS backups stored in S3, manual snapshot limit 100 per Region: consistent with pages inspected.
- Typo in notes: "Microsoft Active Delivery" → Active Directory; "Aurone"-style typos are STYLE (not reported).
