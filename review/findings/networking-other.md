# Triage: Networking & Content Delivery (networking-other.md)

## Summary

- **Section reviewed:** `notes/working/networking-other.md` (Route 53, Global Accelerator, CloudFront, Lambda@Edge/CloudFront Functions, Shield, WAF, API Gateway, Firewall Manager, GraphQL/AppSync)
- **Official exam guide checked:** AWS Certified Solutions Architect – Associate (SAA-C03) Exam Guide, Version 1.1 (https://d1.awsstatic.com/training-and-certification/docs-sa-assoc/AWS-Certified-Solutions-Architect-Associate_Exam-Guide.pdf), downloaded and text-extracted. Still the current guide linked from the certification page.
- **Live verification status:** Live web access available; official pages were fetched and inspected for 14 of 17 findings.
- **Findings by category and status (17 total: 14 CONFIRMED, 2 UNVERIFIED, 1 UNRESOLVED):**
  - ERROR: 4 CONFIRMED (001, 002, 004, 006); 1 UNVERIFIED (016)
  - OUTDATED: 5 CONFIRMED (003, 005, 007, 008, 013); 1 UNVERIFIED (015)
  - INCOMPLETE: 3 CONFIRMED (009, 010, 012)
  - AMBIGUOUS: 2 CONFIRMED (011, 014); 1 UNRESOLVED (017)
- **Exam scope observations (no separate findings):** The exam guide explicitly lists Route 53, CloudFront, Global Accelerator, API Gateway, AppSync, AWS Shield, AWS WAF and AWS Firewall Manager, and "edge networking services" / "DNS" under 3.4. It does not name Lambda@Edge, CloudFront Functions, OpenAPI/Swagger or GraphQL internals; they are at best weakly aligned (broad "edge processing"/"API creation and management" objectives). This is not evidence they are obsolete. Low-priority/exam-scope review only.
- **Important limitations:** Pages not inspected: Global Accelerator–ALB zonal-shift exclusion, DNSSEC details, Route 53 Resolver inbound/outbound text, Traffic Flow $50 price, CloudFront Smooth Streaming. Those claims were left untouched, not confirmed.


## Findings

### FOUND-001 — HTTP API does support custom domains
- **Category:** ERROR
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Amazon API Gateway → REST vs HTTP → API Management table
- **Original claim:** `Custom domains | Yes | No`
- **Issue:** HTTP APIs support custom domain names.
- **Official evidence:** The comparison table lists Custom domains: REST = Yes, HTTP = Yes (links to "Custom domain names for HTTP APIs").
- **Source:** "Choose between REST APIs and HTTP APIs" — https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-vs-rest.html
- **Assessment:** Direct contradiction of the note.
- **Suggested correction:** Change HTTP column to `Yes`.
- **SAA-C03 relevance:** MEDIUM
- **Further action:** None

### FOUND-002 — HTTP API does not support API keys / per-client rate limiting / usage throttling
- **Category:** ERROR
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** REST vs HTTP → API Management table
- **Original claim:** `API keys | Yes | Yes`, `Per-client rate limiting | Yes | Yes`, `Per-client usage throttling | Yes | Yes`
- **Issue:** All three are REST-only.
- **Official evidence:** Same page: API keys, per-client rate limiting, per-client usage throttling = REST Yes, HTTP No. Intro: "Choose REST APIs if you need features such as API keys, per-client throttling, request validation, AWS WAF integration, or private API endpoints."
- **Source:** https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-vs-rest.html
- **Assessment:** Direct contradiction. Likely a high-value exam distinction.
- **Suggested correction:** Set the HTTP column to `No` for those three rows.
- **SAA-C03 relevance:** HIGH
- **Further action:** None

### FOUND-003 — REST APIs now support private integrations with ALB
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** REST vs HTTP → Integrations table
- **Original claim:** `Private integrations with ALB | No | Yes`
- **Issue:** REST API column is now Yes.
- **Official evidence:** Table row "Private integrations with Application Load Balancers": REST = Yes, HTTP = Yes. Cloud Map remains REST No / HTTP Yes (note correct).
- **Source:** https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-vs-rest.html
- **Assessment:** Supported by the current table. Also new rows not in notes: response streaming (REST Yes), developer portal (REST Yes) — optional, low priority.
- **Suggested correction:** REST column → `Yes`.
- **SAA-C03 relevance:** MEDIUM
- **Further action:** None

### FOUND-004 — Route 53 health check quota is 200, not 50
- **Category:** ERROR (OUTDATED if 50 was once the default)
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Health Checks
- **Original claim:** "Can have **up to 50** health checks"
- **Issue:** Current default quota is 200 active health checks per account (adjustable).
- **Official evidence:** Quotas table: "Health checks — 200 active health checks per AWS account".
- **Source:** "Quotas" (Route 53) — https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/DNSLimitations.html
- **Assessment:** Direct. (Interval 30s default / 10s fast was not re-checked; left unchanged.)
- **Suggested correction:** "up to 200 (default quota, can be increased)". Consider noting calculated (chained) checks may monitor up to 255 child checks.
- **SAA-C03 relevance:** LOW (quota trivia)
- **Further action:** None

### FOUND-005 — Lambda@Edge limits table is out of date
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Lambda@Edge vs. CloudFront Functions table (Function duration, Max memory, Max code size); Lambda@Edge section
- **Original claim:** "Viewer request/response = up to 5s Origin = up to 30s"; "128-3008 MB"; "Viewer = 1MB, Origin = 50MB"; CloudFront Functions "< 1 millisecond"
- **Issue:** Current doc: duration up to 30 s for viewer *and* origin events; memory 128 MB (viewer) and 10,240 MB (origin); code package 50 MB for both. Also "Up to 10,000,000 rps" is now "millions of requests per second". Languages/other rows still match (Node.js & Python; CloudFront Functions JS only, 2 MB, 10 KB; geolocation row matches).
- **Official evidence:** Comparison table in the source below.
- **Source:** "Differences between CloudFront Functions and Lambda@Edge" — https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/edge-functions-choosing.html
- **Assessment:** Direct. Also, CloudFront Functions now supports KeyValueStore (Lambda@Edge doesn't) — optional addition, not required.
- **Suggested correction:** Update the three rows to the values above; replace "10,000,000 requests per second or more" with "millions of requests per second". Resolve the `<check>` on "Only Javascript currently supported" — JavaScript confirmed.
- **SAA-C03 relevance:** LOW–MEDIUM (conceptual comparison matters more than exact numbers)
- **Further action:** None

### FOUND-006 — Shield Advanced is $3,000 per month, not per year
- **Category:** ERROR
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** AWS Shield → Shield Advanced
- **Original claim:** "Costs (\$3000 / year)"
- **Issue:** Fee is $3,000 per month.
- **Official evidence:** Pricing examples: "you will pay the AWS Shield Advanced monthly fee of $3,000", plus data transfer out usage fees. Also states Shield Standard is automatically enabled and free.
- **Source:** AWS Shield Pricing — https://aws.amazon.com/shield/pricing/
- **Assessment:** Direct. Not inspected: the 1-year commitment term (do not add without verifying).
- **Suggested correction:** "\$3000 / month (plus data transfer fees)".
- **SAA-C03 relevance:** MEDIUM (cost trade-off: Standard vs Advanced)
- **Further action:** Verify further (commitment term) only if adding it

### FOUND-007 — Zonal shift supported resources/conditions are outdated
- **Category:** OUTDATED
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Zonal Shift → Conditions
- **Original claim:** "Only supported on ALBs and NLBs with cross-zone load balancing turned off"
- **Issue:** ALB and NLB support zonal shift with cross-zone enabled or disabled. Supported resources also include EC2 Auto Scaling groups and EKS. Must be opted in. "Single AZ per load balancer" is still correct.
- **Official evidence:** "Supported resources" lists ASGs, EKS, ALBs and NLBs "with cross-zone load balancing enabled or disabled". ALB page: "You can start a zonal shift for a specific load balancer only for a single Availability Zone."
- **Source:** https://docs.aws.amazon.com/r53recovery/latest/dg/arc-zonal-shift.resource-types.html ; https://docs.aws.amazon.com/r53recovery/latest/dg/arc-zonal-shift.resource-types.app-load-balancers.html
- **Assessment:** First bullet is wrong today. The Global Accelerator exclusion bullet was not found/inspected — leave unchanged and verify if kept. Product name is now "Amazon Application Recovery Controller (ARC)" (note says "Route 53 ARC") — minor.
- **Suggested correction:** Remove the cross-zone restriction; optionally add ASG/EKS and temporary (max 3 days, extendable) nature.
- **SAA-C03 relevance:** LOW–MEDIUM
- **Further action:** Verify further (GA bullet)

### FOUND-008 — Routing policies: there are now 8 (IP-based missing)
- **Category:** OUTDATED / INCOMPLETE
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** Routing Policies → "7 types"
- **Original claim:** "7 types:" (simple, weighted, latency, failover, geolocation, geoproximity, multi-value)
- **Issue:** IP-based routing is the 8th policy. Also multivalue returns up to 8 healthy records, and routing policies can be used in private hosted zones (except IP-based).
- **Official evidence:** "Choosing a routing policy" lists Simple, Failover, Geolocation, Geoproximity, Latency, IP-based, Multivalue answer, Weighted.
- **Source:** https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html
- **Assessment:** Direct.
- **Suggested correction:** "8 types" and add: **IP-based** routing – route based on client IP ranges (CIDR collections) you define.
- **SAA-C03 relevance:** MEDIUM
- **Further action:** None

### FOUND-009 — AWS WAF attachment targets incomplete; WAF Classic reference outdated
- **Category:** INCOMPLETE (WAF); OUTDATED (Firewall Manager "including classic")
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** AWS WAF → "Can be attached to"; AWS Firewall Manager → "AWS WAF (including classic)"
- **Original claim:** WAF attached to CloudFront and ALB only; Firewall Manager manages "AWS WAF (including classic)"
- **Issue:** WAF also protects API Gateway REST API, AppSync GraphQL API, Cognito user pool, App Runner, Verified Access, Amplify. WAF Classic support ended 30 Sep 2025.
- **Official evidence:** WAF developer guide resource list; migration page: "AWS WAF Classic support will end on September 30, 2025."
- **Source:** https://docs.aws.amazon.com/waf/latest/developerguide/waf-chapter.html ; https://docs.aws.amazon.com/waf/latest/developerguide/waf-migrating-from-classic.html
- **Assessment:** The API Gateway REST (not HTTP) and AppSync points link directly to other sections of the notes. Classic claim in Firewall Manager is from the notes; not independently checked on the Firewall Manager docs.
- **Suggested correction:** Add API Gateway (REST), AppSync, Cognito user pools to the list (minimal). Drop "(including classic)" after checking Firewall Manager docs.
- **SAA-C03 relevance:** HIGH (service-selection questions: WAF on API Gateway/ALB/CloudFront)
- **Further action:** Verify further (Firewall Manager "classic")

### FOUND-010 — AppSync authorization types and caching incomplete
- **Category:** INCOMPLETE
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** AWS AppSync → Authorisation types; Caching options
- **Original claim:** Auth types: API key, AWS IAM, Cognito User Pools. Caching: None / Full request / Per-resolver.
- **Issue:** Also supported: AWS Lambda and OpenID Connect. Caching also has an Operation-level mode (newer). "Not serverless/on-demand instance" is consistent with docs (AppSync hosts ElastiCache instances; small…12xlarge).
- **Official evidence:** Authorization page lists API_KEY, AWS_LAMBDA, AWS_IAM, OPENID_CONNECT, AMAZON_COGNITO_USER_POOLS. Caching page lists None, Full request, Per-resolver, Operation level.
- **Source:** https://docs.aws.amazon.com/appsync/latest/devguide/security-authz.html ; https://docs.aws.amazon.com/appsync/latest/devguide/enabling-caching.html
- **Assessment:** Direct. Existing items remain correct.
- **Suggested correction:** Add Lambda and OpenID Connect to auth list; add operation-level caching.
- **SAA-C03 relevance:** MEDIUM
- **Further action:** None

### FOUND-011 — Global Accelerator custom routing endpoints are VPC subnets
- **Category:** AMBIGUOUS
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** AWS Global Accelerator → Types: "Custom Routing - route to specific EC2 instances"
- **Original claim:** "Custom Routing \- route to specific EC2 instances"
- **Issue:** Endpoints are VPC subnets; traffic is directed to specific EC2 instance destinations (IP/port) within them, not other resources. Also the listed standard endpoints (NLB, ALB, EC2, EIP) are consistent with the "endpoints" component list.
- **Official evidence:** "Endpoints for custom routing accelerators are Amazon VPC subnets… You can only direct traffic to EC2 instances in the subnets, not other resources, like load balancers."
- **Source:** https://docs.aws.amazon.com/global-accelerator/latest/dg/about-custom-routing-endpoints.html
- **Assessment:** Note is broadly right but imprecise.
- **Suggested correction:** "Custom Routing – map users to specific EC2 instance/port destinations within VPC subnet endpoints".
- **SAA-C03 relevance:** LOW
- **Further action:** None

### FOUND-012 — CloudFront: no mention of Origin Access Control for S3 origins
- **Category:** INCOMPLETE
- **Verification status:** CONFIRMED
- **Confidence:** HIGH
- **Location:** CloudFront → Components / Origin
- **Original claim:** Origin includes "S3 bucket…" with `S3OriginConfig`; no access-restriction concept
- **Issue:** The standard way to keep an S3 origin private is OAC (OAI is the legacy approach). OAC doesn't apply to S3 website endpoints.
- **Official evidence:** "We recommend that you use OAC instead [of OAI]" (supports all Regions, SSE-KMS, PUT/DELETE); S3 website endpoints must be set up as custom origins and can't use OAC.
- **Source:** https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html
- **Assessment:** Material omission for secure-architecture questions; one or two sentences suffice.
- **Suggested correction:** Add one bullet under Origin on OAC (and that OAI is legacy).
- **SAA-C03 relevance:** HIGH
- **Further action:** None

### FOUND-013 — Route 53 Resolver renamed "Route 53 VPC Resolver"
- **Category:** OUTDATED (terminology)
- **Verification status:** CONFIRMED
- **Confidence:** MEDIUM
- **Location:** Resolver
- **Original claim:** "Route 53 Resolver (formerly .2 Resolver and Amazon DNS Server)"
- **Issue:** Current documentation uses "Route 53 VPC Resolver" (while API/service namespace is still `route53resolver`). Exam wording may use either.
- **Official evidence:** Quotas page headings "Quotas on Route 53 VPC Resolver", "Quotas on Route 53 VPC Resolver endpoints".
- **Source:** https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/DNSLimitations.html
- **Assessment:** Name change confirmed; rest of paragraph not rechecked. Note "to an on-premise network or other VPC" for outbound endpoints is unchanged and unverified.
- **Suggested correction:** "Route 53 VPC Resolver (previously Route 53 Resolver)".
- **SAA-C03 relevance:** LOW
- **Further action:** None

### FOUND-014 — Firewall Manager "$100 per month" is ambiguous
- **Category:** AMBIGUOUS
- **Verification status:** CONFIRMED
- **Confidence:** MEDIUM
- **Location:** AWS Firewall Manager (end)
- **Original claim:** "\$100 per month."
- **Issue:** Charge is $100 per month per policy (plus AWS Config rules and underlying service charges). Notes don't say what the $100 applies to.
- **Official evidence:** Pricing example: "AWS Firewall Manager charges $100 per month for the policy… In addition, AWS Firewall Manager creates two AWS Config rules per policy, per account."
- **Source:** https://aws.amazon.com/firewall-manager/pricing/
- **Assessment:** Per-policy confirmed from the example; per-Region nuance not inspected.
- **Suggested correction:** "\$100 per month per policy (plus AWS Config costs)".
- **SAA-C03 relevance:** LOW
- **Further action:** Verify further (per-Region)

### FOUND-015 — OWASP Top 10 list is the 2017 edition
- **Category:** OUTDATED
- **Verification status:** UNVERIFIED
- **Confidence:** LOW
- **Location:** AWS Web Application Firewall (WAF) → OWASP list
- **Original claim:** 1 Injection … 10 Insufficient logging and monitoring (2017 edition)
- **Issue:** Non-AWS source (OWASP) publishes later editions; AWS documentation wasn't inspected to see which version it references. Per the rules, third-party knowledge isn't proof.
- **Official evidence:** None inspected.
- **Source:** None
- **Assessment:** Candidate only.
- **Suggested correction:** Verify what AWS WAF/AWS managed rule documentation says before changing; may be better to say "OWASP Top 10 (list varies by edition)".
- **SAA-C03 relevance:** LOW (exam rarely requires the list)
- **Further action:** Verify further

### FOUND-016 — Likely typos with technical effect (EC1, ISS)
- **Category:** ERROR (typo)
- **Verification status:** UNVERIFIED
- **Confidence:** MEDIUM
- **Location:** Shield Advanced → "Elastic IP → Amazon EC1"; CloudFront → "ISS Microsoft Smooth Streaming"
- **Original claim:** "Amazon EC1"; "ISS Microsoft Smooth Streaming"
- **Issue:** "EC1" is not an AWS service (probably EC2 instances with Elastic IP); "ISS" is probably IIS. Not checked against an AWS page.
- **Official evidence:** None inspected.
- **Source:** None
- **Assessment:** Flagged for author attention.
- **Suggested correction:** Confirm intent, then correct to EC2 / IIS.
- **SAA-C03 relevance:** LOW
- **Further action:** Verify further

### FOUND-017 — Geoproximity "Only available via Traffic Flow"
- **Category:** AMBIGUOUS
- **Verification status:** UNRESOLVED
- **Confidence:** LOW
- **Location:** Routing Policies → Geo-proximity
- **Original claim:** "Only available via Traffic Flow"
- **Issue:** The geoproximity page says the *maps* are available only with Traffic Flow, and the quotas page lists "Geoproximity records — 30 records that have the same name and type", suggesting geoproximity records can exist outside traffic policies. Evidence doesn't clearly settle whether geoproximity can be created without Traffic Flow. The current page also describes bias as 1–99 / −1 to −99 and Local Zone support.
- **Official evidence:** Statements above.
- **Source:** https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy-geoproximity.html ; https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/DNSLimitations.html
- **Assessment:** Not enough to decide; don't change yet.
- **Suggested correction:** None until the record-creation docs are checked.
- **SAA-C03 relevance:** MEDIUM
- **Further action:** Verify further
