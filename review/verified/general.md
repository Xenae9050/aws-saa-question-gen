# Verification Report: General

- **Notes section:** `notes/working/general.md`
- **Triage report:** `review/findings/general.md`
- **Verification date:** 2026-10-09
- **Exam scope authority:** Current SAA-C03 exam guide on docs.aws.amazon.com (In-Scope and Out-of-Scope Services pages; no version/date shown). The triage's PDF v1.1 was also re-read and **differs** from the live pages (see FOUND-011, FOUND-013).

## 1. Summary table

| Finding ID | Verification outcome | Recommended disposition | Priority | Change since triage? |
| ---------- | -------------------- | ----------------------- | -------- | -------------------- |
| FOUND-001  | CONFIRMED            | CORRECT                 | MEDIUM   | NO                   |
| FOUND-002  | CONFIRMED            | CORRECT                 | LOW      | YES                  |
| FOUND-003  | CONFIRMED            | CORRECT                 | LOW      | NO                   |
| FOUND-004  | CONFIRMED            | CORRECT                 | MEDIUM   | YES                  |
| FOUND-005  | CONFIRMED            | CORRECT                 | LOW      | YES                  |
| FOUND-006  | CONFIRMED            | CORRECT                 | HIGH     | NO                   |
| FOUND-007  | CONFIRMED            | CORRECT                 | HIGH     | NO                   |
| FOUND-008  | CONFIRMED            | CLARIFY                 | MEDIUM   | YES                  |
| FOUND-009  | CONFIRMED            | CORRECT                 | MEDIUM   | YES                  |
| FOUND-010  | PARTIALLY SUPPORTED  | CORRECT                 | MEDIUM   | YES                  |
| FOUND-011  | CONFIRMED            | REMOVE_OR_DEPRIORITISE  | LOW      | YES                  |
| FOUND-012  | CONFIRMED            | CORRECT                 | MEDIUM   | YES                  |
| FOUND-013  | PARTIALLY SUPPORTED  | REMOVE_OR_DEPRIORITISE  | MEDIUM   | YES                  |
| FOUND-014  | CONFIRMED            | CORRECT                 | LOW      | NO                   |
| FOUND-015  | PARTIALLY SUPPORTED  | REMOVE_OR_DEPRIORITISE  | LOW      | YES                  |
| FOUND-016  | CONFIRMED            | CORRECT                 | LOW      | NO                   |
| FOUND-017  | CONFIRMED            | CORRECT                 | MEDIUM   | YES                  |
| FOUND-018  | CONFIRMED            | CORRECT                 | MEDIUM   | NO                   |
| FOUND-019  | CONFIRMED            | CORRECT                 | HIGH     | NO                   |
| FOUND-020  | CONFIRMED            | CORRECT                 | HIGH     | NO                   |
| FOUND-021  | CONFIRMED            | CORRECT                 | HIGH     | NO                   |
| FOUND-022  | PARTIALLY SUPPORTED  | CLARIFY                 | MEDIUM   | YES                  |

No triage finding was wholly rejected. One sub-claim (Lambda scaling in FOUND-022) was rejected and is shown below so the false positive is visible.

## 2. Details for changed findings

### FOUND-002 — CLI "Python is required"

**Change from triage:** Triage inferred "no Python needed" from a version string. AWS now states it explicitly; confidence rises MEDIUM → HIGH.
**Original assessment:** OUTDATED, MEDIUM. True for v1 only; v2 bundles Python (inferred).
**Verified conclusion:** Technically correct for CLI v1 only, but outdated. v2 is current and needs no separate Python.
**Official evidence:** The v2 migration page lists "Python interpreter not needed … doesn't need a separate install of Python. It includes an embedded version." The v2 welcome page says v2 is "available to install only as a bundled installer". The v1 guide states v1 is in maintenance mode.
**Source:**

- Migration guide for the AWS CLI version 2 (What's new in v2) – https://docs.aws.amazon.com/cli/latest/userguide/cliv2-migration-changes.html
- What is the AWS CLI? – https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html
- Configuring settings for the AWS CLI (v1 banner) – https://docs.aws.amazon.com/cli/v1/userguide/cli-chap-configure.html

**Recommended disposition:** CORRECT
**Proposed replacement:** "AWS **CLI** is an executable program. Current version (v2) bundles its own Python; v1 required Python and is in maintenance mode."
**Remaining uncertainty:** None.

### FOUND-004 — STS "global service … single endpoint"

**Change from triage:** Triage said the evidence "directly contradicts" the note. The IAM docs still describe STS as global by default, so the correction should be softer.
**Original assessment:** OUTDATED, HIGH. Regional endpoints recommended; `http` should be `https`.
**Verified conclusion:** Technically correct but outdated and incomplete. The global endpoint is legacy, Regional endpoints exist and are recommended. "All requests go to a single endpoint" is wrong. The URL must be `https`.
**Official evidence:** "AWS recommends using Regional AWS STS endpoints instead of the global endpoint to reduce latency, build in redundancy, and increase session token validity." The global (legacy) endpoint `https://sts.amazonaws.com` is "hosted in a single AWS Region, US East (N. Virginia)". For default-enabled Regions it is now served in the caller's Region. Another IAM page still says "By default, AWS STS is a global service with a single endpoint … However, you can also choose to make AWS STS API calls to endpoints in any other supported Region."
**Source:**

- Manage AWS STS in an AWS Region – https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_enable-regions.html
- Temporary security credentials in IAM ("AWS STS and AWS regions") – https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html

**Recommended disposition:** CORRECT
**Proposed replacement:** "Default endpoint is global (legacy) **https\://sts.amazonaws.com**; Regional endpoints are also available and recommended (lower latency, redundancy)."
**Remaining uncertainty:** None.

### FOUND-005 — Temporary credentials "minutes up to an hour"

**Change from triage:** UNVERIFIED → CONFIRMED.
**Original assessment:** AMBIGUOUS, MEDIUM. Role sessions may exceed one hour; not confirmed.
**Verified conclusion:** Technically wrong as a general limit. Duration is configurable, up to 12 hours for role sessions.
**Official evidence:** IAM: credentials "can be configured to last for anywhere from a few minutes to several hours". AssumeRole `DurationSeconds`: 900 s minimum, up to the role's maximum session duration (1–12 hours; max 43,200 s), default 3,600 s. Role chaining limits the session to one hour.
**Source:**

- Temporary security credentials in IAM – https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html
- AssumeRole API reference (DurationSeconds) – https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html

**Recommended disposition:** CORRECT
**Proposed replacement:** "…only last from **minutes** up to **several hours** (role sessions: default 1 hour, up to 12 hours)."
**Remaining uncertainty:** None. GetSessionToken/GetFederationToken limits were not checked and are not needed.

### FOUND-008 — Personal Health Dashboard / Service Health Dashboard

**Change from triage:** Triage asked whether the Service Health Dashboard contrast should be removed. It should stay but be renamed.
**Original assessment:** OUTDATED, HIGH. Rename to AWS Health Dashboard; verify or remove the Service Health Dashboard paragraph.
**Verified conclusion:** Technically correct but outdated. The account-specific view is now "AWS Health Dashboard – Your account health" (formerly Personal Health Dashboard). The public, account-independent view is "AWS Health Dashboard – Service health". The contrast in the note is still valid conceptually.
**Official evidence:** "You can use the AWS Health Dashboard – Service health to view the health of all AWS services … You don't need to sign in or have an AWS account." Signed-in users are redirected to "Your account health". The exam guide lists "AWS Health Dashboard".
**Source:**

- AWS Health Dashboard (Service health) – https://docs.aws.amazon.com/health/latest/ug/aws-health-dashboard-status.html
- What is AWS Health? – https://docs.aws.amazon.com/health/latest/ug/what-is-aws-health.html
- SAA-C03 In-Scope AWS Services – https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/saa-03-in-scope-services.html

**Recommended disposition:** CLARIFY
**Proposed replacement:** Rename heading to "AWS Health Dashboard (formerly Personal Health Dashboard)". Change the contrast to: "Not to be confused with the public Service health page, which shows general health of AWS services by Region (no sign-in needed)."
**Remaining uncertainty:** The legacy "Service Health Dashboard" name was not found in the current docs. Its replacement by "Service health" is inferred from the page structure.

### FOUND-009 — Root user task list

**Change from triage:** Triage was MEDIUM with several items unchecked. The full current list is now inspected and shows more discrepancies.
**Original assessment:** OUTDATED. Organizations caveat for "Close AWS account"; recheck "Create organization".
**Verified conclusion:** Technically wrong or outdated for four items.
**Official evidence:** The "Tasks that require root user credentials" list (described as the complete list) does **not** include "Create organization" or "Change or cancel AWS Support plan". "Change account settings" is narrower: standalone accounts need root only for email, root password and root access keys; name, contact info, alternate contacts, payment currency and Regions do not. Closing an account needs root only for standalone accounts. Member accounts can be closed from the management/delegated-admin account. Still listed as root-only: restore IAM user permissions, activate IAM access to Billing, view certain tax invoices, GovCloud sign-up, RI Marketplace seller, S3 MFA delete, edit or delete a deny-all S3 bucket policy. Also: "MFA is enforced for root users by default", though the customer must add the device.
**Source:** AWS account root user – "Tasks that require root user credentials" – https://docs.aws.amazon.com/IAM/latest/UserGuide/id_root-user.html#root-user-tasks
**Recommended disposition:** CORRECT
**Proposed replacement:** Narrow "Change account settings" to "Email address, root user password, root user access keys (standalone accounts)". Annotate "Close AWS account" with "(standalone accounts; member accounts via Organizations)". Remove "Create organization" and "Change or cancel AWS Support plan" or mark them unverified. Optionally change "Recommended MFA" to "MFA enforced by default".
**Remaining uncertainty:** The two removals rest on absence from a list AWS calls complete. Confidence MEDIUM.

### FOUND-010 — IAM policy element "Principle"

**Change from triage:** UNVERIFIED → partially verified.
**Original assessment:** AMBIGUOUS, MEDIUM. Misspelling; Principal only in resource-based policies; "orange box" is UI-dependent. Not live-checked.
**Verified conclusion:** Technically wrong: the element is `Principal`, and it is used only in resource-based policies (including role trust policies). It cannot be used in identity-based policies. The "orange box" detail is unresolved.
**Official evidence:** "You must use the Principal element in resource-based policies." "You cannot use the Principal element in an identity-based policy … the principal is implicitly the identity where the policy is attached." The managed-vs-inline page describes AWS managed policies but says nothing about an orange icon.
**Source:**

- AWS JSON policy elements: Principal – https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html
- Managed policies and inline policies – https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html

**Recommended disposition:** CORRECT
**Proposed replacement:** "Principal \- account, user, role or federated user allowed/denied access (resource-based and trust policies only)". Fix the same spelling in the Detective section ("IAM Principles"). No change proposed for "orange box".
**Remaining uncertainty:** "Orange box" could not be verified in documentation. Retain it.

### FOUND-011 — Forecast and Fraud Detector availability

**Change from triage:** Forecast UNVERIFIED → CONFIRMED. Also, both services are no longer in the live exam guide's in-scope list.
**Original assessment:** OUTDATED. Fraud Detector confirmed; Forecast unverified; both in-scope per exam guide v1.1.
**Verified conclusion:** Both are technically correct but outdated: closed to new customers. Neither appears in the current In-Scope list (the PDF v1.1 listed both).
**Official evidence:**

- Fraud Detector: "no longer open to new customers as of November 7, 2025." AWS points to SageMaker, AutoGluon and AWS WAF Fraud Control.
- Forecast: "no longer available to new customers. Existing customers … can continue to use the service as normal."
- The live In-Scope Machine Learning list contains Comprehend, Lex, Polly, Rekognition, SageMaker AI, Textract, Transcribe, Translate. It omits Forecast, Fraud Detector, Kendra and Personalize.

**Source:**

- Amazon Fraud Detector availability change – https://docs.aws.amazon.com/frauddetector/latest/ug/frauddetector-availability-change.html
- What Is Amazon Forecast? (banner) – https://docs.aws.amazon.com/forecast/latest/dg/what-is-forecast.html
- SAA-C03 In-Scope AWS Services – https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/saa-03-in-scope-services.html

**Recommended disposition:** REMOVE_OR_DEPRIORITISE
**Proposed replacement:** Add "(closed to new customers)" to each heading and mark both as optional background. No rewrite of the service descriptions.
**Remaining uncertainty:** Absence from the live list is not proof of exclusion; the list is "non-exhaustive".

### FOUND-012 — Elastic Transcoder / MediaConvert

**Change from triage:** Triage could not verify Elastic Transcoder's lifecycle. It has been discontinued, which is a bigger issue than scope.
**Original assessment:** EXAM SCOPE. ET in-scope, MediaConvert out-of-scope; lifecycle unverified.
**Verified conclusion:** Technically correct but outdated. AWS discontinued Elastic Transcoder on 13 November 2025, so the note should not present it as a usable service. The note's "new version is MediaConvert" is correct as a successor statement. The live exam guide still lists Elastic Transcoder as in-scope and MediaConvert as out-of-scope.
**Official evidence:** "On November 13, 2025, AWS will discontinue support for Amazon Elastic Transcoder. After November 13, 2025, you will no longer be able to access the Amazon Elastic Transcoder console or Amazon Elastic Transcoder resources." The Elastic Transcoder developer guide URL now returns 404.
**Source:**

- Amazon Elastic Transcoder FAQs (notice) – https://aws.amazon.com/elastictranscoder/faqs/
- Amazon Elastic Transcoder Pricing (notice) – https://aws.amazon.com/elastictranscoder/pricing/
- SAA-C03 In-Scope Media Services / Out-of-Scope Media Services – https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/saa-03-in-scope-services.html, https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/saa-03-out-of-scope-services.html

**Recommended disposition:** CORRECT
**Proposed replacement:** Append to the Elastic Transcoder paragraph: "Discontinued 13 Nov 2025; replaced by AWS Elemental MediaConvert." Keep the concept (managed transcoding) because the exam guide still names it. Optionally mark the MediaConvert section "(listed out-of-scope in the exam guide; background)".
**Remaining uncertainty:** The exam guide and service status conflict. Keep the concept note, not the how-to.

### FOUND-013 — Out-of-scope services

**Change from triage:** Part of the triage claim does not hold against the current guide.
**Original assessment:** EXAM SCOPE, HIGH. Exam guide v1.1 lists CodeGuru, Personalize, Cloud9 and MediaConvert as out-of-scope.
**Verified conclusion:** Personalize and MediaConvert remain explicitly out-of-scope. **Cloud9 and CodeGuru are no longer on the current Out-of-Scope list**, though neither is on the In-Scope list. Amazon Q and CodeWhisperer are on neither list. Separately, both Cloud9 and CodeGuru Reviewer have been closed to new use.
**Official evidence:**

- Out-of-Scope Developer Tools (live): CDK, CloudShell, CodeArtifact, CodeBuild, CodeCommit, CodeDeploy, Corretto, FIS, Tools and SDKs. Cloud9 and CodeGuru are not listed. The PDF v1.1 still lists them.
- Out-of-Scope Machine Learning includes Amazon Personalize; Media Services includes MediaConvert.
- CodeGuru Reviewer: "As of November 7, 2025, you can't create new repository associations."
- Cloud9: "no longer available to new customers."

**Source:**

- SAA-C03 Out-of-Scope AWS Services – https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/saa-03-out-of-scope-services.html
- What is Amazon CodeGuru Reviewer? – https://docs.aws.amazon.com/codeguru/latest/reviewer-ug/welcome.html
- What is AWS Cloud9? – https://docs.aws.amazon.com/cloud9/latest/user-guide/welcome.html

**Recommended disposition:** REMOVE_OR_DEPRIORITISE
**Proposed replacement:** Mark Personalize and MediaConvert "(out of scope for SAA-C03; background)". Deprioritise CodeGuru and Cloud9 on SAA relevance (developer tooling, closed or restricted), not on an explicit exam exclusion. No rewrite.
**Remaining uncertainty:** The live guide shows no version or date, so the Cloud9 and CodeGuru removals cannot be dated.

### FOUND-015 — CodeWhisperer / Q Developer tiers

**Change from triage:** UNVERIFIED → PARTIALLY SUPPORTED. The rebrand is supported. The tier numbers and the triage's "security scans" explanation are not.
**Original assessment:** OUTDATED, MEDIUM. CodeWhisperer folded into Q Developer; limits wrong (from background knowledge).
**Verified conclusion:** Technically correct but outdated as a product description. `aws.amazon.com/codewhisperer/` now permanently redirects to the Amazon Q Developer page. Q Developer offers inline completions and security scanning, with a Free tier and a Pro subscription. The "50 users / 500 users per month" limits are not verifiable from current pages, and the Q Developer Free tier lists "50 agentic requests per month". AWS also announced that Q Developer IDE plugin support ends 30 April 2027 (Kiro recommended).
**Official evidence:** Redirect from the CodeWhisperer URL (HTTP 301 to `/q/developer/`). Q Developer docs: "available through a Free tier and the Amazon Q Developer Pro subscription." The pricing page mentions no CodeWhisperer tiers.
**Source:**

- What is Amazon Q Developer? – https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/what-is.html
- Amazon Q Developer pricing – https://aws.amazon.com/q/developer/pricing/
- Amazon Q Developer (end-of-support notice; target of the CodeWhisperer redirect) – https://aws.amazon.com/q/developer/

**Recommended disposition:** REMOVE_OR_DEPRIORITISE
**Proposed replacement:** Replace the CodeWhisperer section with one line: "CodeWhisperer is now part of Amazon Q Developer (Free and Pro tiers)." Not in the exam guide.
**Remaining uncertainty:** Historical CodeWhisperer limits cannot be confirmed from live pages. Do not state what the "50/500" figures measured.

### FOUND-017 — Cognito listed under Directory Service

**Change from triage:** UNVERIFIED → CONFIRMED after reading the full page.
**Original assessment:** ERROR, MEDIUM. Cognito not seen in the Directory Service options.
**Verified conclusion:** Technically wrong. Directory Service directory types are AWS Managed Microsoft AD (Standard/Enterprise/Hybrid), AD Connector and Simple AD. Cognito appears only in the "Which to choose" table, as a recommendation for SaaS developers.
**Official evidence:** "I develop SaaS applications – Use Amazon Cognito if you develop high-scale SaaS applications and need a scalable directory to manage and authenticate your subscribers".
**Source:** What is AWS Directory Service? – https://docs.aws.amazon.com/directoryservice/latest/admin-guide/what_is.html
**Recommended disposition:** CORRECT
**Proposed replacement:** Remove the Cognito bullet from "Offers". Optionally add: "For app user sign-up/sign-in at scale use Amazon Cognito (separate service)."
**Remaining uncertainty:** None.

### FOUND-022 — Lambda scaling and related claims

**Change from triage:** The scaling sub-claim is **rejected**. The ARM and container-image sub-claims are only partly supported.
**Original assessment:** AMBIGUOUS, MEDIUM. Docs show a 500-per-10 s burst, contradicting "1000 … in seconds". ARM and container-image wording unchecked.
**Verified conclusion:**

- **Scaling (REJECTED):** The Lambda docs are internally inconsistent. One illustrative sentence says "a burst increase of 500 concurrency every 10 seconds". The same page's scaling-rate paragraph and the Quotas page say each function can scale by 1,000 execution environments every 10 seconds. The default account concurrency quota is 1,000 per Region (adjustable). The note's "scales … to 1000 … concurrently in seconds" is broadly consistent with these. Only the "functionals" typo and the missing "default quota" qualifier remain.
- **ARM (PARTIALLY SUPPORTED):** Docs say arm64 (Graviton2) achieves "significantly better price and performance". They do not attribute this to "smaller instruction sets".
- **Container image "slower" (UNRESOLVED):** No supporting statement was found. The 10 GB uncompressed image limit is confirmed (zip: 250 MB unzipped).

**Source:**

- Understanding Lambda function scaling / concurrency – https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html
- Lambda quotas – https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html
- Selecting an instruction set architecture – https://docs.aws.amazon.com/lambda/latest/dg/foundation-arch.html

**Recommended disposition:** CLARIFY
**Proposed replacement:**

- Scaling: "…scales automatically up to the account concurrency limit (default 1,000 per Region, can be raised)."
- ARM: "arm64 (Graviton2) typically gives better price-performance."
- Container image: no replacement proposed; retain or drop "but slower" at the author's discretion.

**Remaining uncertainty:** The AWS Lambda docs contradict themselves on the per-function burst rate (500 vs 1,000 per 10 s), so avoid quoting a specific burst figure.

## 3. Unchanged findings

Summary-table rows only. Evidence reviewed (all CONFIRMED, no change from triage):

- FOUND-001: CLI precedence order (command line, then environment variables, … credentials file, then config file) – https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html
- FOUND-003: credentials file is plaintext with `[section]` headers and `name=value` entries, not TOML – https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html
- FOUND-006: "On July 26, 2022, AWS Single Sign-On was renamed to AWS IAM Identity Center"; identity sources are an external IdP, Active Directory and the Identity Center directory – https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html, https://docs.aws.amazon.com/singlesignon/latest/userguide/manage-your-identity-source.html
- FOUND-007: GuardDuty is defined as "a threat detection service". No prevention capability is described. Evidence is definitional; no page states "GuardDuty is not an IPS" – https://docs.aws.amazon.com/guardduty/latest/ug/what-is-guardduty.html
- FOUND-014: Cloud9 closed to new customers – https://docs.aws.amazon.com/cloud9/latest/user-guide/welcome.html
- FOUND-016: Cognito Sync closed to new customers; AppSync/DynamoDB recommended – https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-sync.html
- FOUND-018: `ResultSelector` is the correct field name, and is JSONPath only. JSONata is supported as an alternative query language – https://docs.aws.amazon.com/step-functions/latest/dg/input-output-inputpath-params.html, https://docs.aws.amazon.com/step-functions/latest/dg/state-task.html. The page URL cited in triage (`input-output-resultselector.html`) now redirects to the guide index; the conclusion stands on these pages.
- FOUND-019: KMS keys are protected by "FIPS 140-3 Security Level 3" HSMs; CloudHSM FIPS-mode clusters are 140-2 or 140-3 Level 3. AWS "has replaced the term customer master key (CMK) with … KMS key" (now explicitly sourced). Automatic rotation defaults to yearly, with a custom period and on-demand rotation – https://docs.aws.amazon.com/kms/latest/developerguide/overview.html, https://docs.aws.amazon.com/kms/latest/developerguide/rotate-keys.html, https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateKey.html, https://docs.aws.amazon.com/cloudhsm/latest/userguide/introduction.html
- FOUND-020: Private CA is $400/month (general-purpose) or $50/month (short-lived); ACM-managed certificates carry no extra charge; certificates are regional, and CloudFront requires us-east-1; import is separate from issuance – https://aws.amazon.com/private-ca/pricing/, https://docs.aws.amazon.com/acm/latest/userguide/acm-overview.html
- FOUND-021: Secrets Manager rotates as often as every 4 hours; `rate()` maximum is 999 days (the triage left the upper bound unchecked; it does not change the correction) – https://docs.aws.amazon.com/secretsmanager/latest/userguide/rotate-secrets_schedule.html

## 4. Counts

- CONFIRMED: 18
- PARTIALLY SUPPORTED: 4 (FOUND-010, 013, 015, 022)
- REJECTED: 0 (one sub-claim, Lambda scaling in FOUND-022, rejected)
- UNRESOLVED: 0 (sub-claims unresolved: "orange box" in FOUND-010; container image "slower" in FOUND-022)
- Materially changed vs triage: 12 (FOUND-002, 004, 005, 008, 009, 010, 011, 012, 013, 015, 017, 022)

## 5. Live-verification limitations

- No search engine was available. Official pages were fetched by direct URL. Several JS-heavy pages were read as extracted HTML text (aws.amazon.com product/pricing/FAQ pages and some docs pages).
- The live exam-guide pages show no version or date. The PDF v1.1 differs from them (Forecast, Fraud Detector, Cloud9 and CodeGuru), so exam-scope conclusions follow the live pages. Neither was confirmed to be the newest edition.
- Elastic Transcoder's developer guide is gone (404). Its discontinuation rests on the FAQ and pricing notices.
- Historical CodeWhisperer tier limits and the IAM console "orange box" icon could not be verified.
- GuardDuty "not an IPS" rests on the definition of the service, not an explicit negative statement.
- The Lambda docs are inconsistent with each other on burst scaling.
- The "Create organization" and "Change or cancel AWS Support plan" removals (FOUND-009) rely on absence from the root-tasks list.
- Items in "Items read but not flagged" in the triage (Lex, Polly, Rekognition, Textract, Translate, Comprehend, Device Farm, Batch, etc.) were not re-verified.
