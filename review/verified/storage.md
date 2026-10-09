# Verification: Storage (notes/working/storage.md)

Triage report: `review/findings/storage.md` (FOUND-001 to FOUND-014).
Verified: live inspection of official AWS documentation pages and the SAA-C03 Exam Guide PDF (Version 1.1; PDF metadata last modified 2025-02-07).

## 1. Summary table

| Finding ID | Verification outcome | Recommended disposition | Priority | Change since triage? |
| :--------- | :------------------- | :---------------------- | :------- | :------------------- |
| FOUND-001  | CONFIRMED            | CORRECT                 | HIGH     | NO                   |
| FOUND-002  | CONFIRMED            | CORRECT                 | MEDIUM   | YES                  |
| FOUND-003  | CONFIRMED            | CORRECT                 | LOW      | NO                   |
| FOUND-004  | CONFIRMED            | RETAIN                  | LOW      | NO                   |
| FOUND-005  | CONFIRMED            | CORRECT                 | HIGH     | NO                   |
| FOUND-006  | CONFIRMED            | EXPAND                  | HIGH     | YES                  |
| FOUND-007  | CONFIRMED            | CORRECT                 | HIGH     | NO                   |
| FOUND-008  | PARTIALLY SUPPORTED  | CORRECT                 | MEDIUM   | NO                   |
| FOUND-009  | UNRESOLVED           | INVESTIGATE_FURTHER     | LOW      | NO                   |
| FOUND-010  | CONFIRMED            | CORRECT                 | MEDIUM   | YES                  |
| FOUND-011  | PARTIALLY SUPPORTED  | CLARIFY                 | MEDIUM   | YES                  |
| FOUND-012  | CONFIRMED            | CORRECT                 | MEDIUM   | NO                   |
| FOUND-013  | CONFIRMED            | CORRECT                 | MEDIUM   | NO                   |
| FOUND-014  | PARTIALLY SUPPORTED  | EXPAND                  | LOW      | YES                  |

## 2. Details for changed findings

### FOUND-002 — io2 / io2 Block Express wording and limits

**Change from triage:** The open question ("retired for block express" meaning) is resolved, and the proposed correction is now definite. Validity is unchanged.
**Original assessment:** CONFIRMED, MEDIUM; use-case cell conflicts with the note's own 4,000 MB/s row; "retired" wording unclear; further verification needed.
**Verified conclusion:** As of 30 April 2025 every io2 volume (new and existing) is an io2 Block Express volume, so "io2" and "io2 Block Express" are no longer separate types. The note lists them as two types and says "higher throughput, IOPS and larger capacity" for Block Express as if io2 were lower. The use-case cell ("<1ms latency, >64k IOPS, 1000MB/s") should read >80,000 IOPS or >2,000 MiB/s. The size, 256k IOPS and 4,000 MB/s figures in the table are correct.
**Official evidence:** The io2 Block Express section states "As of April 30, 2025, all new and previously created io2 volumes are io2 Block Express volumes", with up to 64 TiB, 256,000 IOPS (Nitro instances), 4,000 MiB/s, 99.999% durability and average latency under 500 µs. The volume types table lists the use case as "More than 80,000 IOPS or 2,000 MiB/s of throughput".
**Source:** Amazon EBS Provisioned IOPS SSD volumes — https://docs.aws.amazon.com/ebs/latest/userguide/provisioned-iops.html (Considerations; Performance); Amazon EBS volume types — https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volume-types.html (SSD table).
**Recommended disposition:** CORRECT
**Proposed replacement:** Types list: "Io2 (Block Express) - highest performance and durability; all io2 volumes now run on Block Express". Remove the separate "Io2 Block Express" bullet and "(retired for block express)". Use-case cell: "Io2 = sub-ms latency, >80k IOPS or >2,000 MiB/s".
**Remaining uncertainty:** None material. Relevance MEDIUM (SSD volume types are in objective 4.1; exact limits are low-yield).

### FOUND-006 — EFS storage classes / pricing omitted or stale

**Change from triage:** The price figure, unverified at triage, is now checked. The $0.30 figure is the EFS Standard price in AWS's own pricing example, but "starting at $0.30" is misleading because IA and Archive are much cheaper.
**Original assessment:** CONFIRMED omission of storage classes and throughput modes; $0.30/GB unverified.
**Verified conclusion:** Omission confirmed (Standard, IA, Archive; Regional vs One Zone; Elastic/Provisioned Throughput). The EFS FAQ worked example prices EFS Standard at $0.30/GB-month and EFS IA at $0.0165/GB-month. "Starting at $0.30" is therefore wrong as a floor. The Region for the FAQ example is not stated in the text inspected, so treat $0.30 as indicative only.
**Official evidence:** The pricing page lists the three classes and Elastic Throughput with optional Provisioned Throughput. The How-it-works page covers Regional and One Zone file systems. The FAQ pricing example gives the $0.30 and $0.0165 figures.
**Source:** Amazon EFS Pricing — https://aws.amazon.com/efs/pricing/ ; Amazon EFS FAQs (pricing examples) — https://aws.amazon.com/efs/faq/ ; How Amazon EFS works — https://docs.aws.amazon.com/efs/latest/ug/how-it-works.html
**Recommended disposition:** EXPAND
**Proposed replacement:** "Charged by space used; price depends on storage class (Standard, Infrequent Access, Archive) and Region. Standard is about \$0.30 per GB per month in the AWS pricing example; IA and Archive are cheaper but add access/tiering charges. One Zone file systems store data in a single AZ." Optionally mention Elastic Throughput as the default throughput mode (the pricing page describes it as the pay-per-use model, with Provisioned Throughput optional).
**Remaining uncertainty:** Current per-Region prices were not confirmed (dynamic pricing page). Do not hard-code a price without checking.

### FOUND-010 — Transfer Family ports wrong

**Change from triage:** The AS2 port, left unverified at triage, is now verified and the note's value is also inaccurate.
**Original assessment:** CONFIRMED for FTP/FTPS; AS2 443 not verified.
**Verified conclusion:** FTP and FTPS port claims in the note are wrong (control and data are reversed for FTP, and FTPS 990 is not what Transfer Family uses). Transfer Family AS2 servers provide HTTP only, on port 5080. HTTPS (443) is possible only by terminating TLS on a Network or Application Load Balancer in front of the server. SFTP 22 is correct (VPC endpoints can also use 2222, 2223 or 22000).
**Official evidence:** "FTP servers for Transfer Family operate over Port 21 (Control Channel) and Port Range 8192–8200 (Data Channel)." Same wording for FTPS. "AWS Transfer Family AS2 servers currently only provide HTTP transport over port 5080. However, you can terminate TLS on a network or application load balancer…". FTP requires a VPC-hosted internal endpoint and does not encrypt traffic.
**Source:** Create an FTP-enabled server — https://docs.aws.amazon.com/transfer/latest/userguide/create-server-ftp.html ; Create an FTPS-enabled server — https://docs.aws.amazon.com/transfer/latest/userguide/create-server-ftps.html ; Create an SFTP-enabled server — https://docs.aws.amazon.com/transfer/latest/userguide/create-server-sftp.html ; Send and receive AS2 messages ("Receive AS2 messages over HTTPS") — https://docs.aws.amazon.com/transfer/latest/userguide/send-as2-messages.html ; AS2 server limitations — https://docs.aws.amazon.com/transfer/latest/userguide/create-b2b-server.html
**Recommended disposition:** CORRECT
**Proposed replacement:**

- FTP - **21** (control), **8192–8200** (data); unencrypted, VPC-internal only
- SFTP - **22**
- FTPS - **21** (control), **8192–8200** (data)
- AS2 - **5080** (HTTP); HTTPS only via a load balancer in front

**Remaining uncertainty:** None. Relevance MEDIUM (Transfer Family is named in objective 4.1; exact port numbers are lower-yield than protocol-to-use-case mapping).

### FOUND-011 — Migration Hub closed to new customers

**Change from triage:** The exam-scope premise is wrong. The triage said Migration Hub was not named in the exam guide. The SAA-C03 guide (v1.1) appendix lists "AWS Migration Hub", "AWS Application Migration Service" and "AWS Snow Family" under in-scope Migration and Transfer services. Relevance should be MEDIUM, not LOW, and the "consider whether to keep section" suggestion is not supported. The "AMS" abbreviation finding is confirmed and the service name has also changed.
**Original assessment:** CONFIRMED availability change; relevance LOW; suggested reconsidering the section and fixing "AMS".
**Verified conclusion:** Migration Hub has been closed to new customers since 7 November 2025 and AWS Transform is the recommended replacement. Existing customers can continue to use it. It remains listed in the current exam guide, so keep the section and add an availability note. Application Migration Service is abbreviated "MGN", not "AMS", and its documentation is now titled "AWS Transform MGN".
**Official evidence:** "AWS Migration Hub is no longer open to new customers as of November 7, 2025… AWS Transform… is our recommended solution." MGN docs: "What Is AWS Transform MGN? AWS Transform MGN (MGN) automates the migration of physical, virtual, and cloud servers to AWS". Exam guide appendix, Migration and Transfer: "AWS Application Migration Service; AWS Database Migration Service; AWS DataSync; AWS Migration Hub; AWS Snow Family; AWS Transfer Family".
**Source:** AWS Migration Hub availability change — https://docs.aws.amazon.com/migrationhub/latest/ug/migrationhub-availability-change.html ; What Is AWS Transform MGN? — https://docs.aws.amazon.com/mgn/latest/ug/what-is-application-migration-service.html ; SAA-C03 Exam Guide v1.1, Appendix — https://d1.awsstatic.com/training-and-certification/docs-sa-assoc/AWS-Certified-Solutions-Architect-Associate_Exam-Guide.pdf
**Recommended disposition:** CLARIFY
**Proposed replacement:** Keep the section. Change "Application Migration Service (AMS)" to "Application Migration Service (MGN)". Add one line: "Migration Hub is closed to new customers (since 7 Nov 2025); AWS Transform is the recommended replacement."
**Remaining uncertainty:** The exam guide has not been updated for the closure, so a future exam guide revision may drop it. The sub-features (Refactor, Journeys, Discovery Agent) were not individually verified.

### FOUND-014 — Unchecked items (for reference)

**Change from triage:** Several previously unchecked items are now verified.
**Original assessment:** UNRESOLVED; candidates only.
**Verified conclusion:**

- gp3 "up to 20% lower cost per GB than gp2": correct. RETAIN.
- Tape Gateway archive target: confirmed (S3 Glacier Flexible Retrieval or Deep Archive). The note omits it. EXPAND with one line.
- AWS Backup supported-resource list: every service named in the note appears in the AWS Backup feature availability table (EC2, S3, EBS, RDS/Aurora, EFS, FSx, Storage Gateway, DocumentDB, Neptune, Timestream, DynamoDB, virtual machines, SAP HANA on EC2). Note says "such as", so the list is not meant to be exhaustive. RETAIN.
- Tape Gateway "Readability of 30 years": no support found in the Tape Gateway user guide or Storage Gateway FAQ; neither confirmed nor contradicted. UNRESOLVED.
- File Cache description and DataSync target list: not checked in this pass.

**Official evidence:** "gp3 volumes offer a 20 percent lower price per GiB than General Purpose SSD (gp2) volumes." Tape Gateway: "archive backup data in S3 Glacier Flexible Retrieval or S3 Glacier Deep Archive."
**Source:** Amazon EBS General Purpose SSD volumes — https://docs.aws.amazon.com/ebs/latest/userguide/general-purpose.html ; What is Tape Gateway — https://docs.aws.amazon.com/storagegateway/latest/tgw/WhatIsStorageGateway.html ; AWS Backup feature availability — https://docs.aws.amazon.com/aws-backup/latest/devguide/backup-feature-availability.html
**Recommended disposition:** EXPAND
**Proposed replacement:** Add under Tape Gateway: "Archives virtual tapes to S3 Glacier Flexible Retrieval or S3 Glacier Deep Archive." No change to the gp3 or AWS Backup lines.
**Remaining uncertainty:** Tape "30 years" claim, File Cache and DataSync lists were not verified.

## 3. Notes on unchanged findings (no detail required; items worth recording)

- **FOUND-007:** "Standard Value (Default)" is a typo for "standard backup vault"; AWS uses this term when comparing with logically air-gapped vaults. Treat as STYLE.
- **FOUND-008:** Snowball Edge availability and the 210 TB storage-optimized and 28 TB compute-optimized capacities are confirmed. The note's 80 TB and 39.5 TB figures do not appear in current docs. Snowcone and Snowmobile could not be verified: their old documentation and product URLs now redirect to the Snowball Edge pages, and neither the Snowball product page nor the Snowball FAQ mentions them. This is circumstantial, not evidence of discontinuation, so do not edit those lines on this basis. "AWS Snow Family" is in the exam guide appendix and "AWS Snowball Edge" is in objective 4.2.
- **FOUND-009:** No Snowmobile documentation could be inspected; the original claim ("glacier for snowmobile") is neither supported nor contradicted.
- **FOUND-012:** Confirmed. The FSx File Gateway page also states that gateways can be hosted as an AMI in EC2, so the note's hosting line is correct. S3 File Gateway remains the current file gateway.
- **FOUND-013:** Confirmed. Cached volumes 1 GiB–32 TiB (up to 32 volumes, 1,024 TiB total); stored volumes 1 GiB–16 TiB (up to 512 TiB total). The "Compressed" claim for stored volume snapshots was not found on the page inspected.

## 4. Additional observation (not a triage finding)

- EBS table, Provisioned IOPS durability: "Io1 = 99.8%-99.8%" should be 99.8%–99.9% (EBS volume types page).

## 5. Counts and limitations

| Outcome             | Count                               |
| :------------------ | :---------------------------------- |
| CONFIRMED           | 10                                  |
| PARTIALLY SUPPORTED | 3 (FOUND-008, FOUND-011, FOUND-014) |
| REJECTED            | 0                                   |
| UNRESOLVED          | 1 (FOUND-009)                       |
| **Total**           | **14**                              |

Materially changed findings: **5** (FOUND-002, FOUND-006, FOUND-010, FOUND-011, FOUND-014).

Live-verification limitations:

- Snowcone and Snowmobile documentation and product pages redirect to Snowball Edge pages; their current status could not be confirmed.
- EFS and EBS pricing pages render prices dynamically; EFS $0.30 was verified only via the FAQ worked example (Region not stated).
- Tape Gateway "30 years", File Cache and DataSync lists were not verified.
- The exam guide is Version 1.1 with no publication date. A newer guide was not found, but not conclusively ruled out.
