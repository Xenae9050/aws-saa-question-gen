# Verification: Networking & Content Delivery (notes/working/networking-other.md)

Triage report: `review/findings/networking-other.md`. Verified 2026-10-09 against live-fetched official AWS pages. Study notes were not modified.

## 1. Summary table

| Finding ID | Verification outcome | Recommended disposition | Priority | Change since triage? |
| ---------- | -------------------- | ----------------------- | -------- | -------------------- |
| FOUND-001  | CONFIRMED            | CORRECT                 | MEDIUM   | NO                   |
| FOUND-002  | CONFIRMED            | CORRECT                 | HIGH     | NO                   |
| FOUND-003  | CONFIRMED            | CORRECT                 | MEDIUM   | NO                   |
| FOUND-004  | CONFIRMED            | CORRECT                 | LOW      | NO                   |
| FOUND-005  | CONFIRMED            | CORRECT                 | LOW      | YES                  |
| FOUND-006  | CONFIRMED            | CORRECT                 | MEDIUM   | NO                   |
| FOUND-007  | CONFIRMED            | CORRECT                 | LOW      | NO                   |
| FOUND-008  | CONFIRMED            | CORRECT                 | MEDIUM   | NO                   |
| FOUND-009  | CONFIRMED            | CORRECT                 | HIGH     | NO                   |
| FOUND-010  | CONFIRMED            | EXPAND                  | MEDIUM   | NO                   |
| FOUND-011  | CONFIRMED            | CLARIFY                 | LOW      | NO                   |
| FOUND-012  | CONFIRMED            | EXPAND                  | HIGH     | NO                   |
| FOUND-013  | CONFIRMED            | CLARIFY                 | LOW      | YES                  |
| FOUND-014  | CONFIRMED            | CLARIFY                 | LOW      | YES                  |
| FOUND-015  | UNRESOLVED           | CLARIFY                 | LOW      | YES                  |
| FOUND-016  | CONFIRMED            | CORRECT                 | LOW      | YES                  |
| FOUND-017  | CONFIRMED            | CORRECT                 | MEDIUM   | YES                  |

Classification of the confirmed findings:

- Technically wrong: FOUND-001, 002, 004, 006, 016, 017 (the "only via Traffic Flow" claim).
- Technically correct but outdated: FOUND-003, 005, 007, 008, 009, 013.
- Technically correct but incomplete or imprecise: FOUND-010, 011, 012, 014.
- FOUND-015: edition claim could not be settled from AWS sources; separate wording issue found (see below).

Exam-scope source: SAA-C03 Exam Guide v1.1 (https://d1.awsstatic.com/training-and-certification/docs-sa-assoc/AWS-Certified-Solutions-Architect-Associate_Exam-Guide.pdf), downloaded and text-extracted. It names Amazon Route 53, CloudFront, Global Accelerator, API Gateway, AppSync, AWS Shield, AWS WAF and AWS Firewall Manager in the in-scope services list, and Task Statement 3.4 covers "network services with appropriate use cases (for example, DNS)" and edge networking. Lambda@Edge, CloudFront Functions, OpenAPI and GraphQL internals are not named; this is not evidence that they are out of scope.

Items verified without changing the triage assessment (no detail entry needed):

- Official evidence for FOUND-001 to 004, 006 to 012 matches the triage evidence (pages re-fetched: API Gateway REST vs HTTP table, Route 53 Quotas, Shield pricing, ARC zonal shift supported resources and ALB page, Route 53 routing policy, WAF resource list, AppSync authorization and caching, Global Accelerator custom routing endpoints, CloudFront S3 origin access).
- FOUND-006: Shield Advanced 1-year subscription commitment is now also confirmed (Shield pricing page: "requires a 1-year subscription commitment and charges a monthly fee"), so it may be added if wanted.
- FOUND-007: ARC zonal shift supports ASGs, EKS, ALBs and NLBs; "single AZ per load balancer" confirmed for ALB and NLB; shifts are temporary (1 minute to 3 days, extendable). Product name is "Amazon Application Recovery Controller (ARC)".
- FOUND-009: Firewall Manager's chapter intro lists "AWS WAF" with no Classic mention, and the only Classic content is a migration topic ("Migrating AWS WAF Classic Web ACLs in Firewall Manager"). WAF Classic support end date of 30 Sep 2025 has passed. The triage's open item is resolved, so "(including classic)" can be dropped without further checking. The current WAF resource list also includes Amazon Bedrock AgentCore Gateway (not needed for SAA).

## 2. Details for changed findings

### FOUND-005 — Lambda@Edge limits table out of date

**Change from triage:** The proposed correction is wider. The triage said the geolocation row "matches", but the current table gives a different Lambda@Edge value.
**Original assessment:** CONFIRMED, OUTDATED. Duration, memory and code size rows were stale; geolocation row correct.
**Verified conclusion:** The triage's three row updates are correct. In addition, the geolocation/device row is now "No (viewer request and viewer response); Yes (origin request and origin response)". The note reads "Viewer request = No Viewer response, origin request/response = Yes", which says viewer response = Yes. This is the opposite of the current table for viewer response.
**Official evidence:** Comparison table lists: duration up to 30 s for all four Lambda@Edge events; memory 128 MB (viewer) and 10,240 MB (origin); code package 50 MB for viewer and origin; CloudFront Functions scale "up to millions of requests per second", duration "Submillisecond", 2 MB memory, 10 KB size; Lambda@Edge scale still "10,000 requests per second per Region"; CloudFront Functions language is JavaScript (ECMAScript 5.1 compliant), which resolves the `<check>` note. CloudFront Functions also support KeyValueStore (Lambda@Edge does not).
**Source:** Differences between CloudFront Functions and Lambda@Edge — https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/edge-functions-choosing.html (comparison table).
**Recommended disposition:** CORRECT
**Proposed replacement:** Function duration: Lambda@Edge "Up to 30s (all four event types)"; Max memory: "128 MB (viewer) / 10,240 MB (origin)"; Max code + libs size: "50 MB (viewer and origin)"; Scale for CloudFront Functions: "millions of requests per second"; Geolocation and device data access: Lambda@Edge "Viewer request/response = No, origin request/response = Yes"; remove `<check>` (JavaScript only).
**Remaining uncertainty:** None.

### FOUND-013 — Route 53 Resolver renamed "Route 53 VPC Resolver"

**Change from triage:** Confidence raised MEDIUM to HIGH, and the reason for the rename is now known.
**Original assessment:** CONFIRMED (name seen only in quota page headings), MEDIUM confidence.
**Verified conclusion:** The service's documentation title is "What is Route 53 VPC Resolver?" with an explicit rename note. The inbound/outbound endpoint descriptions in the notes match the current text.
**Official evidence:** "Route 53 VPC Resolver was previously called Route 53 Resolver, but was renamed when Route 53 Global Resolver was introduced." Inbound endpoints allow DNS queries "to your VPC from your on-premises network or another VPC"; outbound endpoints allow queries "from your VPC to your on-premises network or another VPC". The VPC+2 address is described as the Resolver address.
**Source:** What is Route 53 VPC Resolver? — https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resolver.html (introductory note and endpoint list).
**Recommended disposition:** CLARIFY
**Proposed replacement:** "Route 53 VPC Resolver (previously Route 53 Resolver; formerly .2 Resolver and Amazon DNS Server) is a DNS server..." Rest of paragraph unchanged. Do not confuse with the separate Route 53 Global Resolver.
**Remaining uncertainty:** None. Exam wording may still use the old name.

### FOUND-014 — Firewall Manager "$100 per month" is ambiguous

**Change from triage:** The per-Region nuance, left open in triage, is now resolved. There is also a Shield Advanced exception.
**Original assessment:** CONFIRMED, AMBIGUOUS. $100 per policy per month plus AWS Config costs; per-Region not checked.
**Verified conclusion:** The fee is $100 per month per policy, charged per Region. Customers subscribed to Shield Advanced do not pay the Firewall Manager policy fee (they still pay the AWS Config rule charges).
**Official evidence:** Pricing page: "protection policies are priced with a monthly fee per region"; "AWS Firewall Manager charges $100 per month for the policy… creates two AWS Config rules per policy, per account"; "For AWS Shield Advanced customers, AWS Firewall Manager protection policy is included at no additional charge." The underlying-service charges (WAF web ACLs/rules, Network Firewall endpoints) are billed separately.
**Source:** AWS Firewall Manager Pricing — https://aws.amazon.com/firewall-manager/pricing/ (pricing overview and Example 7).
**Recommended disposition:** CLARIFY
**Proposed replacement:** "$100 per month per policy per Region (plus AWS Config and underlying service charges; policy fee waived for Shield Advanced customers)."
**Remaining uncertainty:** The Firewall Manager developer guide intro says charges are "for the underlying services, such as AWS WAF and AWS Config", which reads differently from the pricing page. The pricing page was treated as authoritative for cost. The pricing-table figure itself was only visible through the worked example, not the dynamic price table.

### FOUND-015 — OWASP Top 10 list is the 2017 edition

**Change from triage:** Remains unresolved on the edition question, but the evidence shows a different issue: the note overstates what WAF covers. The recommended action changes from "verify" to "clarify wording".
**Original assessment:** UNVERIFIED, LOW. Possible outdated edition; no AWS evidence inspected.
**Verified conclusion:** AWS documentation does not publish or pin an OWASP edition. It only says the Core rule set protects against "some of the high risk and commonly occurring vulnerabilities described in OWASP publications such as OWASP Top 10". The note's list does match the 2017 edition (XXE, insecure deserialisation, insufficient logging), but the OWASP site could not be fetched, so edition currency was not checked, and OWASP is not an AWS authority. The statements "WAF … for OWASP top 10 protection" (CloudFront section) and "protect web apps from attacks covered in the OWASP top 10" imply full coverage, which AWS does not claim.
**Official evidence:** WAF Core rule set (CRS) description; the Application Load Balancer integrations page repeats the same "OWASP publications such as OWASP Top 10" wording.
**Source:** Baseline rule groups — https://docs.aws.amazon.com/waf/latest/developerguide/aws-managed-rule-groups-baseline.html (Core rule set); Integrations for your Application Load Balancer — https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-integrations.html (AWS WAF section).
**Recommended disposition:** CLARIFY
**Proposed replacement:** Optional minimal wording: "Protects web apps from common attacks, including some of those in the OWASP Top 10 (list below is the 2017 edition)". Keep the list; no AWS source requires changing it.
**Remaining uncertainty:** Whether a later OWASP edition should replace the list is an OWASP question, not verified here. Low SAA relevance.

### FOUND-016 — Likely typos with technical effect (EC1, ISS)

**Change from triage:** UNVERIFIED to CONFIRMED, with official text identifying the intended terms.
**Original assessment:** UNVERIFIED, MEDIUM. "EC1" and "ISS" probably typos for EC2 and IIS.
**Verified conclusion:** Both are errors. Shield Advanced protects EC2 instances through association to Elastic IP addresses, and NLBs through Elastic IP associations. CloudFront documents "Microsoft Smooth Streaming" and Microsoft IIS as the web server. "ISS Microsoft Smooth Streaming" is wrong; the doc does not use "ISS".
**Official evidence:** Shield Advanced resource list: "Amazon EC2 instances, through association to Amazon EC2 Elastic IP addresses" and "Network Load Balancers, through associations to Amazon EC2 Elastic IP addresses". CloudFront VOD page: "Configure video on demand for Microsoft Smooth Streaming… web server that runs Microsoft IIS".
**Source:** List of AWS resources that AWS Shield Advanced protects — https://docs.aws.amazon.com/waf/latest/developerguide/ddos-advanced-summary-protected-resources.html ; Deliver video on demand with CloudFront — https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/on-demand-video.html
**Recommended disposition:** CORRECT
**Proposed replacement:** "Amazon EC2" (under Elastic IP) and "Microsoft Smooth Streaming" (drop "ISS", or write "IIS Smooth Streaming" only if the author meant the IIS server).
**Remaining uncertainty:** Author intent for "ISS" (the CloudFront feature is "Smooth Streaming", independent of an IIS origin).

### FOUND-017 — Geoproximity "Only available via Traffic Flow"

**Change from triage:** UNRESOLVED to CONFIRMED. Console record-creation documentation settles the question.
**Original assessment:** UNRESOLVED. The maps are Traffic Flow-only but the evidence did not show whether records can be created without it.
**Verified conclusion:** Geoproximity records can be created with the standard Create record workflow (routing policy "Geoproximity") and the API (`GeoProximityLocation` on a resource record set). Only the geoproximity _map_ in the visual editor is tied to Traffic Flow. The note's statement is therefore wrong as written.
**Official evidence:** "Creating records by using the Route 53 console" links to "Values specific for geoproximity records" and "geoproximity alias records", where the step is: Routing policy — "Choose Geoproximity", then endpoint location (coordinates, AWS Region or Local Zone Group) and bias. The geoproximity page notes "The maps are available only with Traffic Flow" and the Traffic Flow page lists the geoproximity map as a visual-editor feature. Quotas page: "Geoproximity records — 30 records that have the same name and type". Bias range is -99 to 99.
**Source:** Creating records by using the Amazon Route 53 console — https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resource-record-sets-creating.html ; Values specific for geoproximity records — https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resource-record-sets-values-geoprox.html ; Geoproximity routing — https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy-geoproximity.html
**Recommended disposition:** CORRECT
**Proposed replacement:** "Traffic Flow provides a visual map; geoproximity records can also be created directly" (replace "Only available via Traffic Flow"). Optional: note bias is -99 to +99 and endpoints can be an AWS Region, Local Zone Group or lat/long.
**Remaining uncertainty:** None for the record-creation question. This was not tested by creating a record.

## 3. Counts and limitations

**Outcome counts (17 findings):** CONFIRMED 16, PARTIALLY SUPPORTED 0, REJECTED 0, UNRESOLVED 1 (FOUND-015).

**Materially changed since triage:** 6 (FOUND-005, 013, 014, 015, 016, 017).

**Live-verification limitations:**

- Zonal shift: the bullet "Not supported when using ALB as an accelerator endpoint in AWS Global Accelerator" was not found in the ARC, ALB, NLB or Global Accelerator pages inspected. It is neither confirmed nor refuted; leave unchanged and verify before relying on it.
- Not inspected: Route 53 health check interval (30 s default, 10 s fast).
- Spot-checked only, no finding raised: Traffic Flow "$50/month per policy record" (Route 53 pricing: "$50.00 per policy record per month") and the DNSSEC KSK statement (DNSSEC page: each KSK is based on a customer-managed KMS key).
- OWASP site could not be fetched (redirect refused); no non-AWS source was used for FOUND-015.
- Shield Advanced price page shows $3,000 monthly fee in worked examples; no dynamic price table was inspected.
- Firewall Manager price table is dynamic; $100 per policy was confirmed through a worked example.
