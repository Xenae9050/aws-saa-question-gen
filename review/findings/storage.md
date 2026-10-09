# Triage: Storage (notes/working/storage.md)

## Summary

- Section reviewed: `notes/working/storage.md` (EBS, HDD/RAID/SSD, EFS, FSx, File Cache, AWS Backup, Snow Family, Transfer Family, Migration Hub, DataSync, Storage Gateway)
- Official exam guide checked: AWS Certified Solutions Architect – Associate (SAA-C03) Exam Guide, **Version 1.1** (no publication date in the PDF). Relevant objectives: 1.2/2.1 (Transfer Family), 3.1 (hybrid storage, S3/EFS/EBS), 4.1 (FSx, EFS, EBS HDD/SSD types, backup strategies, DataSync/Transfer Family/Storage Gateway), 4.2 (Snowball Edge as hybrid compute). Migration Hub, Snowcone, Snowmobile and File Cache are **not named** in the guide.
- Live verification status: Live web access worked. Most findings CONFIRMED against inspected AWS pages. FOUND-008 (Snowcone/Snowmobile part) is only partly verified; FOUND-009 and FOUND-014 are unresolved.
- Findings: 14 total. By category: ERROR 4, OUTDATED 5, INCOMPLETE 2, AMBIGUOUS 3. By status: CONFIRMED 12 (FOUND-008 only partly), UNRESOLVED 2 (FOUND-009, FOUND-014).
- Important limitations: Snowcone/Snowmobile and some AWS availability-change pages redirected to index pages and could not be inspected. EFS regional price ($0.30/GB) was not visible on the pricing page fetched. AWS Backup supported-resource list and Tape Gateway "30 years" claim were not checked. No exam-scope removal conclusions made.

## Findings

### FOUND-001 — EBS table: gp3 limits outdated
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** EBS comparison table (General Purpose SSD column) and Types list ("Gp3 - up to 20% lower cost per GB than gp2")
- **Original claim:** Volume size "1GB - 16TB"; Max IOPS "16KB I/O"; Max throughput "gp2=250MB/s gp3=1000MB/s"
- **Issue:** The table merges gp2 and gp3. gp3 is now 1 GiB–64 TiB, 80,000 IOPS (25.6 KiB I/O), 2,000 MiB/s; gp2 remains 16 TiB / 16,000 IOPS / 250 MiB/s. The Max IOPS cell gives only an I/O size, no number. Units are GiB/TiB/MiB/s in AWS docs.
- **Official evidence:** Volume types table lists gp3 "1 GiB - 64 TiB", "80,000 (25.6 KiB I/O)", "2,000 MiB/s"; gp2 "1 GiB - 16 TiB", "16,000 (16 KiB I/O)", "250 MiB/s".
- **Source:** Amazon EBS volume types — https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volume-types.html
- **Assessment:** Directly contradicts note for gp3 throughput and size; the note's IOPS cell is incomplete.
- **Suggested correction:** Split gp2/gp3 in the three rows (gp2: 16 TiB, 16k IOPS, 250 MiB/s; gp3: 64 TiB, 80k IOPS, 2,000 MiB/s). Note the "20% lower cost" claim was not checked (see FOUND-014).
- **SAA-C03 relevance:** HIGH — EBS SSD/HDD types are named in objective 4.1.
- **Further action:** None

### FOUND-002 — io2 / io2 Block Express wording and limits
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** MEDIUM
- **Location:** Types list "Io2 - more durable than io1 (retired for block express)"; table Provisioned IOPS column
- **Original claim:** "Io2 = <1ms latency, >64k IOPS, 1000MB/s throughput"; "Io2 = 4GB-64TB"; "Io2 = 256k"
- **Issue:** The current docs describe a single "io2 Block Express" type (4 GiB–64 TiB, 256,000 IOPS, 4,000 MiB/s, 99.999% durability); the use-case cell (1,000 MB/s) conflicts with the note's own max throughput row (4,000 MB/s) and AWS says use io2 Block Express for >80,000 IOPS or >2,000 MiB/s. The "retired for block express" phrase is unclear. Latency is stated as consistent sub-millisecond, average under 500 µs.
- **Official evidence:** Volume types table: io2 Block Express "4 GiB - 64 TiB", "256,000", "4,000 MiB/s", "99.999% durability"; io1 "4 GiB - 16 TiB", "64,000", "1,000 MiB/s". Footnote: 256,000 IOPS requires Nitro instances.
- **Source:** https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volume-types.html
- **Assessment:** Internal inconsistency confirmed; exact "retired" meaning not verifiable from page.
- **Suggested correction:** Use case cell: "io2 Block Express = sub-ms latency, >80k IOPS or >2,000 MiB/s". Reword "retired for block express" (e.g. "io2 now runs on Block Express") after checking the io2 page.
- **SAA-C03 relevance:** MEDIUM
- **Further action:** Verify further (io2 vs Block Express wording)

### FOUND-003 — Multi-attach and NVMe reservations
- **Category:** AMBIGUOUS
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Table rows "EBS multi-attach", "NVMe reserve"
- **Original claim:** NVMe reserve: "Io1 = No Io2 = Yes"
- **Issue:** AWS lists NVMe reservations as supported for both io1 and io2. Multi-attach "Yes" for Provisioned IOPS is correct (io1/io2 only) but unqualified.
- **Official evidence:** Table: NVMe reservations — io2 Block Express "Supported", io1 "Supported"; Multi-attach — gp "Not supported", Provisioned IOPS "Supported".
- **Source:** https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volume-types.html
- **Assessment:** The io1 "No" is contradicted.
- **Suggested correction:** NVMe reserve: Io1 = Yes, Io2 = Yes. (Low exam value; could be removed.)
- **SAA-C03 relevance:** LOW
- **Further action:** None

### FOUND-004 — Magnetic and HDD rows (verified correct; minor wording)
- **Category:** AMBIGUOUS
- **Verification status:** CONFIRMED
- **Confidence:** MEDIUM
- **Location:** Types list; table Magnetic column; "Throughput Optimised HHD"
- **Original claim:** "Magnetic - previous generation HDD"; "Data very infrequently accessed"
- **Issue:** Values for st1/sc1 (125 GiB–16 TiB, 500/250 IOPS, 500/250 MiB/s, no boot) and magnetic (1 GiB–1 TiB, 40–200 IOPS, 40–90 MiB/s, bootable) match AWS. Only note: durability row for magnetic "N/A" is fine; the table body "HHD" typos are STYLE and not reported. No change needed.
- **Official evidence:** HDD and Previous generation tables on the volume types page.
- **Source:** https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volume-types.html
- **Assessment:** Included so reviewers know it was checked. Retain.
- **Suggested correction:** None
- **SAA-C03 relevance:** HIGH
- **Further action:** None

### FOUND-005 — EFS mount targets and VPC statement
- **Category:** ERROR
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Elastic File System (EFS)
- **Original claim:** "EFS creates multiple mount targets in all VPC subnets." / "(must be in same VPC)"
- **Issue:** One mount target per Availability Zone (one subnet per AZ if there are several), not every subnet. A file system has mount targets in only one VPC at a time, but on-premises servers can mount via Direct Connect/VPN. One Zone file systems have a single mount target.
- **Official evidence:** "You can create one mount target in each Availability Zone… If there are multiple subnets in an Availability Zone… you create a mount target in one of the subnets." "An EFS file system can have mount targets in only one VPC at a time." On-premises mounting via Direct Connect or Site-to-Site VPN is supported.
- **Source:** How Amazon EFS works — https://docs.aws.amazon.com/efs/latest/ug/how-it-works.html
- **Assessment:** The "all subnets" claim is wrong; "same VPC" is correct as stated but incomplete w.r.t. on-premises access.
- **Suggested correction:** "EFS uses one mount target per AZ (in one subnet of that AZ). Mount targets are in a single VPC; on-premises servers can mount via Direct Connect or VPN."
- **SAA-C03 relevance:** HIGH
- **Further action:** None

### FOUND-006 — EFS storage classes / pricing omitted or stale
- **Category:** INCOMPLETE
- **Verification status:** CONFIRMED
- **Confidence:** MEDIUM
- **Location:** EFS — "Charged by space used starting at $0.30 per GB per month."
- **Original claim:** quoted above
- **Issue:** EFS now has three storage classes (Standard, Infrequent Access, Archive), Regional vs One Zone, and Elastic/Provisioned Throughput; none are mentioned, and these matter for cost-optimisation questions. The $0.30/GB figure could not be confirmed on the pricing page fetched (pricing is region- and class-dependent).
- **Official evidence:** Pricing page lists EFS Standard, EFS IA and EFS Archive; Elastic Throughput (optional Provisioned Throughput); Regional/One Zone file systems documented in How it works page.
- **Source:** Amazon EFS Pricing — https://aws.amazon.com/efs/pricing/
- **Assessment:** Omission confirmed; price figure unverified.
- **Suggested correction:** Add one line listing classes (Standard, IA, Archive; One Zone variants) and that price varies by class/Region; drop or re-verify the $0.30 figure.
- **SAA-C03 relevance:** HIGH — objective 4.1 (storage tiering/cost).
- **Further action:** Verify further (price figure)

### FOUND-007 — AWS Backup: immutability and vault wording
- **Category:** ERROR
- **Verification status:** CONFIRMED
- **Confidence:** MEDIUM
- **Location:** AWS Backup
- **Original claim:** "AWS Backups are immutable to avoid tampering." ; "Standard Value (Default) - initial storage place"
- **Issue:** Backups are not immutable by default; immutability comes from Vault Lock (governance or compliance mode) — the note elsewhere correctly mentions Vault Lock. Also "Standard Value" appears to be "Standard vault" (typo, unverified; vault types per docs: backup vault, logically air-gapped vault). Vault Lock modes (governance vs compliance, grace time) are omitted.
- **Official evidence:** "Vaults locked in governance mode can have the lock removed by users with sufficient IAM permissions. Vaults locked in compliance mode cannot be deleted once the grace time expires…"
- **Source:** AWS Backup Vault Lock — https://docs.aws.amazon.com/aws-backup/latest/devguide/vault-lock.html ; Backup vaults — https://docs.aws.amazon.com/aws-backup/latest/devguide/vaults.html
- **Assessment:** The unqualified "immutable" claim is misleading.
- **Suggested correction:** "Backups can be made immutable using Vault Lock (governance or compliance mode)."
- **SAA-C03 relevance:** HIGH — backup strategies, 4.1.
- **Further action:** None

### FOUND-008 — Snow Family: availability and device specs outdated
- **Category:** OUTDATED
- **Verification status:** CONFIRMED (Snowball Edge); UNRESOLVED (Snowcone, Snowmobile)
- **Confidence:** HIGH (Snowball Edge)
- **Location:** AWS Snow Family
- **Original claim:** "Storage optimised (80/210TB) / Compute optimised (39.5TB)"; Snowcone and Snowmobile listed as current
- **Issue:** Snowball Edge is no longer available to new customers. Current docs list only Storage Optimized 210 TB and Compute Optimized (28 TB NVMe), not 80 TB or 39.5 TB. Snowcone redirects to the Snowball page and its status could not be confirmed; Snowmobile status not inspected.
- **Official evidence:** "AWS Snowball Edge is no longer available to new customers. New customers should explore AWS DataSync… AWS Data Transfer Terminal… AWS Outposts." Config table: storage-optimized 210 TB NVMe; compute-optimized 28 TB NVMe.
- **Source:** AWS Snowball Edge device hardware information — https://docs.aws.amazon.com/snowball/latest/developer-guide/device-differences.html
- **Assessment:** Availability change confirmed. Exam guide v1.1 still names "AWS Snowball Edge" under hybrid compute (objective 4.2), so keep it but flag status — availability ≠ exam removal.
- **Suggested correction:** Add a note that Snowball Edge is closed to new customers; update capacities. Verify Snowcone and Snowmobile status before editing those lines.
- **SAA-C03 relevance:** MEDIUM — still referenced in exam guide.
- **Further action:** Verify further (Snowcone/Snowmobile)

### FOUND-009 — Snow Family: "glacier for snowmobile"
- **Category:** AMBIGUOUS
- **Verification status:** UNRESOLVED
- **Confidence:** LOW
- **Location:** AWS Snow Family intro
- **Original claim:** "Data delivered to S3 (or glacier for snowmobile)."
- **Issue:** Candidate only; Snowmobile import target (S3, then lifecycle to Glacier) could not be inspected.
- **Official evidence:** None inspected.
- **Source:** None
- **Assessment:** Not enough evidence.
- **Suggested correction:** Verify against Snowmobile documentation or drop.
- **SAA-C03 relevance:** LOW
- **Further action:** Verify further

### FOUND-010 — Transfer Family ports wrong
- **Category:** ERROR
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** AWS Transfer Families — Common ports
- **Original claim:** "FTP - 20 (control commands), 21 (data transfer)"; "FTPS - 990"
- **Issue:** Control/data are reversed in the note (traditionally 21 = control, 20 = data). Transfer Family FTP and FTPS use port 21 (control) and 8192–8200 (data); FTPS 990 is not what Transfer Family uses. SFTP 22 (also 2222/2223/22000 for VPC endpoints). FTP/FTPS endpoints must be VPC hosted (FTP internal-only). AS2 port 443 not verified.
- **Official evidence:** "FTP servers for Transfer Family operate over Port 21 (Control Channel) and Port Range 8192–8200 (Data Channel)." Same wording for FTPS. SFTP "operate over port 22."
- **Source:** Create an FTP-enabled server — https://docs.aws.amazon.com/transfer/latest/userguide/create-server-ftp.html ; Create an FTPS-enabled server — https://docs.aws.amazon.com/transfer/latest/userguide/create-server-ftps.html ; Create an SFTP-enabled server — https://docs.aws.amazon.com/transfer/latest/userguide/create-server-sftp.html
- **Assessment:** Direct contradiction.
- **Suggested correction:** FTP: 21 (control) + 8192–8200 (data). SFTP: 22. FTPS: 21 + 8192–8200. Also note FTP is unencrypted and VPC-internal only.
- **SAA-C03 relevance:** MEDIUM
- **Further action:** Verify further (AS2 port)

### FOUND-011 — Migration Hub closed to new customers
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** AWS Migration Hub
- **Original claim:** "Single place to discover existing servers, plan migrations and track their status…"; "Application Migration Service (AMS)"
- **Issue:** Migration Hub stopped accepting new customers on 7 Nov 2025; AWS Transform is the recommended replacement. Migration Hub is not named in exam guide v1.1 (not an exam-scope removal). "AMS" is a confusing abbreviation (AWS Managed Services uses it); Application Migration Service is usually MGN (abbreviation not verified here). Whole subsection is questionable for SAA study value.
- **Official evidence:** "AWS Migration Hub has stopped accepting new customers as of November 7, 2025… AWS Transform… is our recommended solution." Lists Strategy Recommendations, Journeys, Orchestrator.
- **Source:** AWS Migration Hub availability change — https://docs.aws.amazon.com/migrationhub/latest/ug/migrationhub-availability-change.html
- **Assessment:** Availability change confirmed.
- **Suggested correction:** Add availability note; fix abbreviation; consider whether to keep section (low SAA focus).
- **SAA-C03 relevance:** LOW
- **Further action:** Review exam-scope interpretation

### FOUND-012 — FSx File Gateway no longer available
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Storage Gateway → File Gateway → "FSx - store data in Windows File Server"
- **Original claim:** quoted above
- **Issue:** FSx File Gateway is closed to new customers; AWS points to direct access to FSx for Windows File Server. Also the note says File Gateway deploys "on AMI in EC2" under each; hosting detail otherwise fine. Docs also list Nutanix AHV and VMware Cloud on AWS as hosts (INCOMPLETE, minor).
- **Official evidence:** "Amazon FSx File Gateway is no longer available to new customers. Existing customers… can continue to use the service normally."
- **Source:** What is Amazon FSx File Gateway — https://docs.aws.amazon.com/filegateway/latest/filefsxw/what-is-file-fsxw.html
- **Assessment:** Confirmed.
- **Suggested correction:** Mark FSx File Gateway as closed to new customers; keep S3 File Gateway (SMB/NFS to S3) as the current file gateway.
- **SAA-C03 relevance:** MEDIUM — Storage Gateway named in 4.1.
- **Further action:** None

### FOUND-013 — Volume Gateway cached volume size
- **Category:** ERROR
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Storage Gateway → Cached volumes → "1GB-32GB"
- **Original claim:** "1GB-32GB"
- **Issue:** Cached volumes are 1 GiB to 32 TiB (up to 32 volumes, 1 PiB per gateway). Stored volumes 1 GiB–16 TiB (up to 512 TiB total) are correct. Snapshots stored as EBS snapshots, incremental — correct. "Compressed" not verified.
- **Official evidence:** "Cached volumes can range from 1 GiB to 32 TiB in size… up to 32 volumes for a total maximum storage volume of 1,024 TiB." "Stored volumes can range from 1 GiB to 16 TiB."
- **Source:** How Volume Gateway works — https://docs.aws.amazon.com/storagegateway/latest/vgw/StorageGatewayConcepts.html
- **Assessment:** Direct contradiction (GB vs TB).
- **Suggested correction:** "1GiB–32TiB".
- **SAA-C03 relevance:** MEDIUM
- **Further action:** None

### FOUND-014 — Unchecked items (for reference)
- **Category:** INCOMPLETE
- **Verification status:** UNRESOLVED
- **Confidence:** LOW
- **Location:** Various
- **Original claim:** gp3 "up to 20% lower cost per GB than gp2"; Tape Gateway "Readability of 30 years"; AWS Backup supported-services list; File Cache description; DataSync target list
- **Issue:** Not verified in this pass; no evidence of error either. Tape Gateway docs confirm archive to S3 Glacier Flexible Retrieval/Deep Archive (note does not say this; worth adding in one line). Typos "iSCI", "Hard Disk", "HHD" are STYLE and not reported.
- **Official evidence:** Tape Gateway page states tapes archive to Glacier Flexible Retrieval or Deep Archive.
- **Source:** What is Tape Gateway — https://docs.aws.amazon.com/storagegateway/latest/tgw/WhatIsStorageGateway.html
- **Assessment:** Candidates only.
- **Suggested correction:** Verify gp3 pricing claim on EBS pricing page; add Glacier archive target to Tape Gateway.
- **SAA-C03 relevance:** MEDIUM
- **Further action:** Verify further

## Not flagged
- HDD/RAID/SSD hardware background: correct, general knowledge, not AWS-specific (low exam value; objective 4.1 mentions HDD/SSD volume types only).
- EBS "replicated within AZ", DataSync protocols/targets, File Cache: no issue found at this depth.
