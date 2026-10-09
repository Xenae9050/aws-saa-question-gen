# Triage: General

## Summary

- Section reviewed: `notes/working/general.md` (935 lines, read in full)
- Official exam guide checked: AWS Certified Solutions Architect – Associate (SAA-C03) Exam Guide, **Version 1.1** (PDF text extracted and inspected, https://d1.awsstatic.com/training-and-certification/docs-sa-assoc/AWS-Certified-Solutions-Architect-Associate_Exam-Guide.pdf). The certification landing page was reachable but returned almost no content, so I could not independently confirm 1.1 is the newest version.
- Live verification status: PARTIAL. Official AWS docs/pages were fetched and inspected for the findings marked CONFIRMED. Findings marked UNVERIFIED rest on candidates from the notes where I could not retrieve a supporting official page.
- Findings: 22 total (ERROR 6, OUTDATED 11, AMBIGUOUS 3, EXAM SCOPE 2, INCOMPLETE 0)
  - CONFIRMED: 18 (some with partially unverified sub-points: FOUND-009, 011, 012, 019, 022)
  - UNVERIFIED: 4 (FOUND-005, 010, 015, 017)
  - UNRESOLVED: 0
- Important limitations:
  - No search engine was available; I fetched known official URLs directly. Several pages (Elastic Transcoder docs, Forecast/CodeGuru availability notices, Q Developer/CodeWhisperer tiers) were missing or gave no usable text.
  - Not every sentence was researched; Lex, Polly, Rekognition, Textract, Translate, Comprehend, Personalize, Device Farm, Batch, LDAP/Directory background were read but nothing material was flagged.
  - Several findings are typos/terminology slips (e.g. `>`/`<`, "Principle") with HIGH confidence from the notes themselves plus a doc cross-check.

## Findings

### FOUND-001 — CLI precedence operator typo
- **Category:** ERROR
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** AWS API > CLI
- **Original claim:** "CLI **parameters** \> **env** vars \< **config** files."
- **Issue:** Inconsistent operators; the second `<` reverses the order of env vars and config files.
- **Official evidence:** Precedence list: command line options first, then environment variables, then assume-role/web identity, credentials file, custom process, config file, container credentials.
- **Source:** AWS CLI v1 User Guide, "Configuring settings for the AWS CLI" – https://docs.aws.amazon.com/cli/v1/userguide/cli-chap-configure.html
- **Assessment:** Env vars override config files, so the order should be params > env vars > config files.
- **Suggested correction:** "CLI **parameters** \> **env** vars \> **config** files."
- **SAA-C03 relevance:** MEDIUM (CLI is in the in-scope list)
- **Further action:** None

### FOUND-002 — CLI "is a Python program (Python is required)"
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** MEDIUM
- **Location:** AWS API > CLI
- **Original claim:** "AWS **CLI** is a **Python** executable program (Python is required)"
- **Issue:** True for v1 (pip install). The current install guide for v2 uses standalone installers; the `aws --version` output shows a bundled Python.
- **Official evidence:** v2 install guide shows installer-based installs (macOS/Windows/Linux) and `aws-cli/2.27.41 Python/3.11.6`. The v1 guide states v1 is in maintenance mode and recommends migrating to v2.
- **Source:** https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html ; https://docs.aws.amazon.com/cli/v1/userguide/cli-chap-configure.html
- **Assessment:** Evidence shows Python is bundled in v2, supporting that a separate Python install is not required. (I did not find an explicit "no Python required" sentence; this is my inference.)
- **Suggested correction:** Note that v2 is current and bundles its own Python; v1 needs Python.
- **SAA-C03 relevance:** LOW
- **Further action:** None

### FOUND-003 — Credentials file is not TOML
- **Category:** ERROR
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** AWS API > Keys
- **Original claim:** "**\~/.aws/credentials** (TOML format)"
- **Issue:** The file uses INI-style sections.
- **Official evidence:** "Section names are enclosed in brackets [ ] … entries take the form `setting_name=value`", plaintext files.
- **Source:** AWS CLI User Guide, "Configuration and credential file settings" – https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html
- **Assessment:** Format described is INI-style (the docs don't use the word "INI"; the label is my interpretation), not TOML.
- **Suggested correction:** "(INI-style format)"
- **SAA-C03 relevance:** LOW
- **Further action:** None

### FOUND-004 — STS described as a single global endpoint
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** AWS API > STS
- **Original claim:** "Global service, all requests go to single endpoint http\://sts.amazonaws.com"
- **Issue:** AWS recommends Regional STS endpoints; the global endpoint is described as legacy. Also `http` should be `https`.
- **Official evidence:** "AWS recommends using Regional AWS STS endpoints instead of the global endpoint to reduce latency, build in redundancy, and increase session token validity."
- **Source:** IAM User Guide, "Manage AWS STS in an AWS Region" – https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_enable-regions.html
- **Assessment:** Directly contradicts "all requests go to a single endpoint".
- **Suggested correction:** "Has a global (legacy) endpoint and Regional endpoints; Regional endpoints are recommended."
- **SAA-C03 relevance:** MEDIUM
- **Further action:** None

### FOUND-005 — STS temporary credentials duration
- **Category:** AMBIGUOUS
- **Verification status:** UNVERIFIED
- **Confidence:** MEDIUM
- **Location:** Temporary Security Credentials
- **Original claim:** "only last from **minutes** up to an **hour**"
- **Issue:** Role session duration can exceed one hour (configurable role max session up to 12 hours, as I recall). Exact limits not confirmed in this triage.
- **Official evidence:** Temporary-credentials page fetched but no matching duration text extracted.
- **Source:** https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html
- **Assessment:** Cannot confirm.
- **Suggested correction:** Verify against AssumeRole `DurationSeconds` docs.
- **SAA-C03 relevance:** LOW
- **Further action:** Verify further

### FOUND-006 — AWS SSO renamed
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** "AWS Single-Sign on (SSO)"
- **Original claim:** "## AWS Single-Sign on (SSO)"
- **Issue:** Renamed AWS IAM Identity Center (26 July 2022). The exam guide lists "AWS IAM Identity Center (AWS Single Sign-On)".
- **Official evidence:** "On July 26, 2022, AWS Single Sign-On was renamed to AWS IAM Identity Center."
- **Source:** https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html ; Exam Guide v1.1 appendix
- **Assessment:** Direct. Identity-source list ("AWS SSO") should read "Identity Center directory".
- **Suggested correction:** Rename heading to "AWS IAM Identity Center (formerly AWS SSO)"; first identity source = "Identity Center directory".
- **SAA-C03 relevance:** HIGH
- **Further action:** None

### FOUND-007 — GuardDuty is not an IPS
- **Category:** ERROR
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Amazon Guard Duty
- **Original claim:** "acting as both an intrusion detection system (IDS) and intrusion protection system (IPS)"
- **Issue:** GuardDuty is a threat detection (findings) service; it does not block traffic. Data-source list also incomplete.
- **Official evidence:** "GuardDuty is a threat detection service that continuously monitors, analyzes, and processes…"; foundational sources: CloudTrail management events, VPC flow logs, DNS logs; additional protection plans (S3, EKS, malware, RDS, Lambda etc.).
- **Source:** https://docs.aws.amazon.com/guardduty/latest/ug/what-is-guardduty.html
- **Assessment:** Docs describe detection only, no prevention. Absence of "IPS" wording is the evidence; treating it as an error is high-confidence from SAA knowledge (blocking is done by e.g. Network Firewall/WAF).
- **Suggested correction:** Remove "and intrusion protection system (IPS)"; optionally note protection plans (S3, EKS, malware).
- **SAA-C03 relevance:** HIGH
- **Further action:** None

### FOUND-008 — Personal Health Dashboard renamed/merged
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** AWS Personal Health Dashboard
- **Original claim:** "## AWS Personal Health Dashboard … Not to be confused with the Service Health Dashboard"
- **Issue:** Current name is AWS Health Dashboard (exam guide also uses this name). The doc describes a single dashboard powered by the AWS Health API; I did not find explicit text on the Service Health Dashboard retirement.
- **Official evidence:** "All customers can use the AWS Health Dashboard, powered by the AWS Health API." Exam guide lists "AWS Health Dashboard".
- **Source:** https://docs.aws.amazon.com/health/latest/ug/what-is-aws-health.html ; Exam Guide v1.1
- **Assessment:** Name is outdated. Whether the separate "Service Health Dashboard" paragraph is still valid needs checking.
- **Suggested correction:** Rename to AWS Health Dashboard; verify/remove the Service Health Dashboard contrast.
- **SAA-C03 relevance:** MEDIUM
- **Further action:** Verify further (Service Health Dashboard paragraph)

### FOUND-009 — Root user task list
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** MEDIUM
- **Location:** AWS Account Root User
- **Original claim:** "Close AWS account", "Create organization", "Cannot be limited (except by … SCP)", "Recommended MFA"
- **Issue:** Root-only task list has changed; in Organizations, management/delegated-admin accounts can close member accounts and update their root email/name/contact/Regions, and member accounts have no root credentials by default. "Create organization" is not shown as root-only in the page extract I inspected.
- **Official evidence:** "With AWS Organizations, you can close the member accounts centrally…"; "Member accounts … have no root user credentials by default."
- **Source:** https://docs.aws.amazon.com/IAM/latest/UserGuide/root-user-tasks.html ; https://docs.aws.amazon.com/IAM/latest/UserGuide/id_root-user.html
- **Assessment:** Supports adding the Organizations caveat to "Close AWS account" and changing account settings. I did not confirm each remaining item (e.g. MFA delete) against the page.
- **Suggested correction:** Add "(standalone accounts; member accounts can be managed centrally via Organizations)"; recheck "Create organization".
- **SAA-C03 relevance:** MEDIUM
- **Further action:** Verify further

### FOUND-010 — IAM policy element "Principle" and Principal applicability
- **Category:** AMBIGUOUS
- **Verification status:** UNVERIFIED
- **Confidence:** MEDIUM
- **Location:** IAM > Policies
- **Original claim:** "Principle \- account, user, role or federated user to apply to"; "Managed … Have orange box next to name"
- **Issue:** Element is spelled "Principal" and appears only in resource-based/trust policies, not identity-based policies. "Orange box" icon detail is UI-dependent. Not live-checked.
- **Official evidence:** None inspected.
- **Source:** None
- **Assessment:** Candidate only.
- **Suggested correction:** Verify against IAM policy elements reference.
- **SAA-C03 relevance:** MEDIUM
- **Further action:** Verify further

### FOUND-011 — ML services: Forecast and Fraud Detector closed to new customers
- **Category:** OUTDATED
- **Verification status:** CONFIRMED (Fraud Detector); UNVERIFIED (Forecast)
- **Confidence:** HIGH (Fraud Detector); LOW (Forecast)
- **Location:** Amazon Forecast; Amazon Fraud Detector
- **Original claim:** (service descriptions without availability caveats)
- **Issue:** Fraud Detector page: "Amazon Fraud Detector is no longer accepting new customers. … explore Amazon SageMaker, AutoGluon, and AWS WAF." Forecast page did not show a notice in what I could extract.
- **Official evidence:** Fraud Detector product page availability notice.
- **Source:** https://aws.amazon.com/fraud-detector/ (links to https://docs.aws.amazon.com/frauddetector/latest/ug/frauddetector-availability-change.html)
- **Assessment:** Fraud Detector confirmed. Forecast not confirmed here; both remain listed in the SAA-C03 exam guide in-scope list.
- **Suggested correction:** Add an availability note for Fraud Detector; verify Forecast.
- **SAA-C03 relevance:** MEDIUM (still in the exam guide appendix)
- **Further action:** Verify further (Forecast)

### FOUND-012 — Elastic Transcoder / MediaConvert exam scope and status
- **Category:** EXAM SCOPE
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** AWS Elastic Transcoder; AWS Elemental Media Convert
- **Original claim:** "New version of Elastic Transcoder is called AWS Elemental MediaConvert."
- **Issue:** Exam guide v1.1 lists Elastic Transcoder as in scope and Elemental MediaConvert as out of scope. Elastic Transcoder's current service status (end of support?) could not be verified (docs URL 404).
- **Official evidence:** Exam guide appendix: "Media Services: Amazon Elastic Transcoder, Amazon Kinesis Video Streams" (in-scope); "AWS Elemental MediaConvert" in out-of-scope list.
- **Source:** Exam Guide v1.1 (URL in Summary)
- **Assessment:** Exam-scope conclusion is explicit. Do not infer the services are obsolete from scope alone.
- **Suggested correction:** Mark the MediaConvert section as out-of-scope (optional background). Verify Elastic Transcoder lifecycle status separately.
- **SAA-C03 relevance:** HIGH
- **Further action:** Verify further (Elastic Transcoder lifecycle)

### FOUND-013 — Out-of-scope services in the notes
- **Category:** EXAM SCOPE
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Amazon CodeGuru; Amazon Personalize; Cloud9 mention under Amazon Q Developer; Amazon Q / CodeWhisperer
- **Original claim:** (whole sections)
- **Issue:** Exam guide v1.1 out-of-scope list includes Amazon CodeGuru, Amazon Personalize, AWS Cloud9, AWS Elemental MediaConvert. Amazon Q and CodeWhisperer are not named in either list (weak/uncertain alignment; no explicit coverage).
- **Official evidence:** Out-of-scope list in the Exam Guide appendix.
- **Source:** Exam Guide v1.1 (URL in Summary)
- **Assessment:** Candidates for trimming, subject to the author's decision. Scope is not the same as correctness.
- **Suggested correction:** Mark as out-of-scope/optional rather than deleting.
- **SAA-C03 relevance:** HIGH
- **Further action:** Review exam-scope interpretation

### FOUND-014 — Cloud9 no longer available to new customers
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Amazon Q > Amazon Q Developer integrations ("Cloud9")
- **Original claim:** "Integrated into: … Cloud9"
- **Issue:** Cloud9 is closed to new customers.
- **Official evidence:** "AWS Cloud9 is no longer available to new customers. Existing customers … can continue to use the service as normal."
- **Source:** https://docs.aws.amazon.com/cloud9/latest/user-guide/welcome.html
- **Assessment:** Direct.
- **Suggested correction:** Remove Cloud9 from the integration list, or annotate.
- **SAA-C03 relevance:** LOW (out of scope)
- **Further action:** None

### FOUND-015 — CodeWhisperer is now part of Amazon Q Developer; tier numbers wrong
- **Category:** OUTDATED
- **Verification status:** UNVERIFIED
- **Confidence:** MEDIUM
- **Location:** Amazon CodeWhisperer
- **Original claim:** "Individual … 50 users / month; Professional … 500 users / month"
- **Issue:** From my background knowledge, CodeWhisperer was folded into Amazon Q Developer (Free tier / Pro) and the limits referred to security scans per month, not users. The fetched Q Developer docs confirmed Free tier + Pro subscription and Slack/Teams chat applications but did not mention CodeWhisperer.
- **Official evidence:** Q Developer page: "available through a Free tier and the Amazon Q Developer Pro subscription."
- **Source:** https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/what-is.html
- **Assessment:** Partial support for the tier rename only.
- **Suggested correction:** Verify, then replace the CodeWhisperer section with a pointer to Q Developer or delete (out of scope).
- **SAA-C03 relevance:** LOW
- **Further action:** Verify further

### FOUND-016 — Cognito Sync closed to new customers
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Amazon Cognito > Methods
- **Original claim:** "Cognito Sync \- Syncs user data and preferences across all devices"
- **Issue:** No longer open to new customers; AWS points to AppSync/DynamoDB.
- **Official evidence:** "Amazon Cognito Sync is no longer open to new customers. … explore AWS AppSync and DynamoDB."
- **Source:** https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-sync.html
- **Assessment:** Direct.
- **Suggested correction:** Annotate as legacy; AppSync is the replacement.
- **SAA-C03 relevance:** LOW
- **Further action:** None

### FOUND-017 — Cognito listed as an AWS Directory Service offering
- **Category:** ERROR
- **Verification status:** UNVERIFIED
- **Confidence:** MEDIUM
- **Location:** Amazon Directory Service > Offers
- **Original claim:** "Amazon Cognito \- integrate signup and sign-in into web apps"
- **Issue:** The Directory Service "What is" page I inspected lists AD Connector and AWS Managed Microsoft AD (and others) and contained no Cognito entry. Cognito is a separate identity service. Simple AD status not checked.
- **Official evidence:** Directory Service admin guide page text matched AD Connector and Managed Microsoft AD, with no Cognito match.
- **Source:** https://docs.aws.amazon.com/directoryservice/latest/admin-guide/what_is.html
- **Assessment:** Absence is supportive but not a full read of the page; marking UNVERIFIED.
- **Suggested correction:** Remove Cognito from the list (it is already covered in its own section).
- **SAA-C03 relevance:** MEDIUM
- **Further action:** Verify further

### FOUND-018 — Step Functions: "ResultsSelector" misspelled
- **Category:** ERROR
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Step Functions > Inputs & Outputs
- **Original claim:** "ResultsSelector \- change a state's result before the ResultPath is applied"
- **Issue:** Field name is `ResultSelector`.
- **Official evidence:** AWS doc page is named `input-output-resultselector` (page fetched). 
- **Source:** https://docs.aws.amazon.com/step-functions/latest/dg/input-output-resultselector.html
- **Assessment:** Direct (spelling). Also, ASL now supports JSONata as an alternative to JSONPath (see ASL page), which the note's "uses JSONPath" statement omits.
- **Suggested correction:** Rename to `ResultSelector`; optionally add "JSONata is also supported".
- **SAA-C03 relevance:** MEDIUM
- **Further action:** None

### FOUND-019 — KMS: CMK renamed; FIPS level and rotation
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Key Management Service (KMS); Customer Master Key (CMK); CloudHSM
- **Original claim:** "FIPS 140-2 level 3 complaint, vs level 2 for multi-tenant"; "Customer Master Key (CMK)"; "enable-key-rotation"
- **Issue:** (a) KMS keys are protected by FIPS 140-3 Security Level 3 HSMs, so the "level 2 for KMS" contrast is stale. (b) "CMK" is now "KMS key". (c) Automatic rotation is every year by default but has a configurable period and on-demand rotation.
- **Official evidence:** "…keys you create in AWS KMS are protected by FIPS 140-3 Security Level 3"; "By default … generates new cryptographic material … every year. You can also specify a custom rotation-period … on-demand rotation."
- **Source:** https://docs.aws.amazon.com/kms/latest/developerguide/overview.html ; https://docs.aws.amazon.com/kms/latest/developerguide/rotate-keys.html
- **Assessment:** (a) and (c) confirmed from the pages. (b) is not shown in the text I extracted (my background knowledge: renamed in 2021); the same pages use "KMS key" throughout.
- **Suggested correction:** Replace the level 2/3 contrast with the real CloudHSM differentiator (single-tenant, customer-controlled keys); add "(now called KMS key)"; add one line on rotation options.
- **SAA-C03 relevance:** HIGH (Exam Guide: "Rotating encryption keys and renewing certificates")
- **Further action:** None

### FOUND-020 — ACM: private certificates mis-described
- **Category:** ERROR
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Amazon Certificate Manager (ACM) > Certificate types
- **Original claim:** "Private \- imported certificates … Costs (\$400/month\!)"; heading "Amazon Certificate Manager"
- **Issue:** Imported third-party certificates are a separate option; the $400 figure is the AWS Private CA monthly charge (general-purpose mode), $50 for short-lived mode. Service name is AWS Certificate Manager. ACM docs also state there is no additional charge for certificates managed by ACM, and certificates can now be exported/used on EC2 (ACME) in some cases.
- **Official evidence:** "$400 per private CA per month for general-purpose mode; $50 per private CA per month for short-lived certificate mode"; ACM overview: "You are not subject to an additional charge for SSL/TLS certificates that you manage with ACM"; can issue directly or import.
- **Source:** https://aws.amazon.com/private-ca/pricing/ ; https://docs.aws.amazon.com/acm/latest/userguide/acm-overview.html
- **Assessment:** Direct.
- **Suggested correction:** "Private – issued by AWS Private CA (CA charged monthly, e.g. $400 general-purpose); imported certificates are a separate option." Rename to AWS Certificate Manager. Add that CloudFront requires certificates in us-east-1.
- **SAA-C03 relevance:** HIGH
- **Further action:** None

### FOUND-021 — Secrets Manager rotation interval
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Secrets Manager
- **Original claim:** "Intervals range from 30 days \- 365 days"
- **Issue:** Rotation uses cron()/rate() schedules with a rotation window; can rotate as often as every four hours.
- **Official evidence:** "You can rotate a secret as often as every four hours within a rotation window as small as one hour."
- **Source:** https://docs.aws.amazon.com/secretsmanager/latest/userguide/rotate-secrets_schedule.html
- **Assessment:** The 30–365 day range is contradicted. I did not find the upper bound in the extracted text.
- **Suggested correction:** "Rotation is configured with a schedule (rate/cron) – as frequently as every four hours; managed rotation also exists for some secrets."
- **SAA-C03 relevance:** HIGH
- **Further action:** None

### FOUND-022 — Lambda scaling and related imprecision
- **Category:** AMBIGUOUS
- **Verification status:** CONFIRMED (scaling); UNVERIFIED (others)
- **Confidence:** MEDIUM
- **Location:** AWS Lambda
- **Original claim:** "scales automatically to 1000 functionals concurrently in seconds"; "ARM is more efficient due to having smaller instruction sets"; Container image "…but slower"
- **Issue:** Per-function scaling is described in terms of burst rate. Other items: ARM statement is not a precise explanation (price-performance); container images have a larger package limit (10 GB, from background knowledge, not checked).
- **Official evidence:** "A function using on-demand concurrency can experience a burst increase of 500 concurrency every 10 seconds, or by 5,000 requests per second every 10 seconds, whichever happens first."
- **Source:** https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html
- **Assessment:** The doc scaling rate differs from the "1000 … in seconds" claim; I could not reconcile the older account-level figures. Mark the scaling statement as needing rewording.
- **Suggested correction:** Reword to "scales automatically; concurrency limited per account/Region and by burst scaling rate"; verify ARM/container-image wording.
- **SAA-C03 relevance:** HIGH
- **Further action:** Verify further

## Items read but not flagged
Lex, Polly, Rekognition, Textract, Translate, Comprehend, Device Farm, Batch, Auto Scaling, Amplify (Gen 2 mention only), Service Catalog, Artifact, Detective, LDAP/Directory Service background, CloudHSM basics. No material issues identified; limits quoted for Kendra, Polly lexicons and Batch were not verified.
