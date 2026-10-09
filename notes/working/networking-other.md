## Route53

Domain Name Service (**DNS**) with integrations with AWS services.  
Can:

- Register & manage domains
- Create record sets on domains
- Implement complex traffic flows
- Monitor records via Health Checks
- Resolve VPCs outside of AWS

**Traffic flow** is a UI workflow supporting creation of sophisticated routing configurations as well as versioning (\$50/month per policy record).

### Hosted Zone

Hosted Zone in Route 53 is a container for record sets, scoped to route traffic for specific domains (e.g. recordname.app.saas) or subdomains (e.g. app.recordname).  
Two types:

- **Public** \- how to route inbound traffic from the internet
- **Private** \- how to route traffic from within an Amazon VPC

### Records

Record Sets are collections of records determining where to send traffic. Changed in batch via the API, with CREATE, DELETE and UPSERT operations. Alias records route traffic to specific AWS endpoints dynamically (reference will survive changes in resource IP). Using alias records is recommended when routing to AWS resources like:

- **CloudFront** (d1234567abcdef8.cloudfront.net)
- **Elastic Beanstalk** env (example.elasticbeanstalk.com)
- **Elastic Load Balancer** (example-1.us-east-2.elb.amazonaws.com)
- **S3** website endpoint (s3-website.us-east-2.amazonaws.com)
- Resource record set (www\.example.com)
- **VPC** endpoint (example.us-east-2.vpc2.amazonaws.com)
- **API** **gateway** endpoint custom regional API (d-abcde-1234.execute-api.us-west-2.amazonaws.com)

### Routing Policies

7 types:

- **Simple** routing (default)
  - Multiple IP addresses per record
  - Traffic routed to one of IP addresses at random
- **Weighted** routing
  - Multiple IP addresses per record
  - Relative weight per address
  - Random routing with frequency correlating with relative weights
- **Latency-based** routing
  - Routing based on lowest-latency region
  - Requires latency resource record for each application-hosting resource in each region
  - Example: 2 copies of 1 application backed by ALB, hosted in different regions and always routed to lower latency of the two as seen by the user
  - Usually proximity-based but not in principle
- **Failover** routing
  - Route 53 will monitor health checks on primary route
  - Will begin routing to secondary location when health checks on primary fail
- **Geolocation** routing
  - Directs traffic based on location (e.g. all north america traffic to us-east-1)
- **Geo-proximity** routing
  - Only available via Traffic Flow
  - Gives different regions different biases (effectively relative ranges)
  - Traffic routed based on location according to which bias-adjusted region they are located in
- **Multi-value answer** routing
  - Same as simple routing but with added health check on targets

### Health Checks

Checks the correct functioning and health of AWS endpoints. Can have **up to 50** health checks for endpoints within, or linked to, an AWS account. Checks performed **every 30s** by default, can be reduced to **10s**. If unhealthy, can initiate a failover or create CloudWatch alarm to alert. Can also chain health checks together by monitoring others in a health check.

### Resolver

Route 53 Resolver (formerly .2 Resolver and Amazon DNS Server) is a DNS server allowing the resolution of DNS queries between an on-premise DNS resolver and a VPC. **Inbound** resolver endpoints allow DNS queries to the VPC from an on-premise network or other VPC. **Outbound** resolver endpoints allow DNS queries from the VPC to an on-premise network or other VPC.

### DNSSEC

Domain Name System Security Extensions (DNSSEC) are a suite of extension specifications by the Internet Engineering Task Force (IETF) for securing data exchanged in the Domain Name System (DNS) in Internet Protocol (IP) networks. DNSSEC signing allows DNS resolvers to validate that a DNS response came from Route 53 and has not been tampered with. It’s important to enable DNSSEC on your domains so others cannot impersonate them. Involves signing with KSK key.

### Zonal Shift

Capability in Amazon Route 53 Application Recovery Controller (ARC) that shifts a load balancer resource away from an impaired availability zone towards a healthy one.  
Conditions:

- Only supported on Application Load Balancers (**ALB**s) and Network Load Balancers (**NLB**s) with cross-zone load balancing turned off
- **Not** supported when using ALB as an accelerator endpoint in AWS Global Accelerator
- Only for **single AZ** per load balancer

### Route 53 Profiles

Profiles let you manage and apply Route 53 DNS-related configurations across different VPCs and AWS accounts.  
You can associate:

- Private hosted zones
- Resolver rules
- DNS firewall rule groups

## AWS Global Accelerator

Can find the optimal path from an end user to one of your web servers. Deployed within **Edge Locations** so traffic is sent here, not directly to the web application. Has a speed comparison tool.  
Types:

- **Standard** \- automatically route to nearest healthy endpoint
- **Custom** Routing \- route to specific EC2 instances

Components:

- **Listeners** (e.g. TCP 3000\) \- listen for traffic on a specific port and sends to an endpoint group
- **Endpoint Groups** \- collection of Endpoints within a specific region. A Traffic Dial can be used to balance traffic loads
- **Endpoints** \- resources to send traffic to. Can be:
  - Network Load Balancer (NLB)
  - Application Load Balancer (ALB)
  - EC2 instance
  - Elastic IP address

## CloudFront

A Content Delivery Network (**CDN**) is a distributed network of servers that provide web pages or other content to users based on geographical location, the origin of the web page and a content delivery server. Essentially caching across a distributed network strategically for service speed.  
CloudFront is a CDN that can be used to deliver:

- Static content
- Dynamic content
- Video streaming
- Web sockets

Components:

- **Origin** \- location where all original files are stored (e.g. S3 bucket, EC2 instance, ELB or Route 53\)
  - Domain name \- e.g. ‘mybucket.s3.amazonaws.com’
  - Origin path (optional) \- e.g. ‘/accounts’
  - S3OriginConfig
  - CustomOriginConfig (non-s3)
- **Edge Location** \- compute located strategically close to end user where copies are cached
- **Regional Cache** \- computed located in broad geographical regions to speed up requests for edge locations
- **Distribution** \- collection of edge locations and regional caches that defines how cached content should behave

CloudFront can:

- Be fronted with AWS WAF for OWASP top 10 protection
- Stream videos on demand using ISS Microsoft Smooth Streaming

## Lambda@Edge

Lambda functions to override the behavior of requests and responses operating via CloudFront.  
Function types (lifecycle order):

1. **Viewer Request** \- when CloudFront receives a request from a viewer

   Use cases:

- Redirect HTTP \-\> HTTPS (CloudFront could just do this)
- Inspect cookies for user authentication
- Modify headers for A/B testing

2. **Origin Request** \- before CloudFront forwards a request to the origin

Use cases:

- Rewrite URLs for SEO or routing
- Inject headers for origin authentication
- Selective content serving based on user-agent (e.g. mobile vs. desktop)

3. **Origin Response** \- when CloudFront receives a response from the origin

Use cases:

- Modify headers to control caching
- Update URLs in HTML for versioning
- Customise error responses from origin

4. **Viewer Response** \- before CloudFront forwards a response to the viewer

   Use cases:

- Add security headers (e.g. CSP, HSTS)
- Set cookies for client-side tracking
- Customise error messages

Functions are deployed at **regional** edge cache level. Supported languages are Python and Node.js.

## CloudFront Functions

Lightweight edge functions for high-scale, latency-sensitive CDN customisations. **Cheaper** and **faster**, but **more limited** than, Lambda@Edge functions.  
Functions (lifecycle order):

1. **Viewer Request** \- when CloudFront receives a request from a viewer
2. **Viewer Response** \- Before CloudFront returns the response to the viewer

Functions are deployed at **edge locations**. Only Javascript currently supported \<check\>.  
Use cases:

- Cache key normalisation
- Header manipulation
- Status code modification and body generation
- URL redirects or rewrites
- Request authorisation

## Lambda@Edge vs. CloudFront Functions

| Category                                     | Lambda@Edge functions                                                    | CloudFront Functions                    |
| -------------------------------------------- | ------------------------------------------------------------------------ | --------------------------------------- |
| Scale                                        | Up to 10,000 request per second per region                               | 10,000,000 requests per second or more  |
| Function duration                            | Viewer request/response \= up to 5s Origin request/response \= up to 30s | \< 1 millisecond                        |
| Max memory                                   | 128-3008 MB                                                              | 2 MB                                    |
| Max code \+ libs size                        | Viewer request/response \= 1MB Origin request/response \= 50MB           | 10 KB                                   |
| Network, file system and request body access | Yes                                                                      | No                                      |
| Geolocation and device data access           | Viewer request \= No Viewer response, origin request/response \= Yes     | Yes                                     |
| Can build & test in CloudFront               | No                                                                       | Yes                                     |
| Function logging & metrics                   | Yes                                                                      | Yes                                     |
| Pricing                                      | Costs per request & function duration                                    | Costs per request (free tier available) |

## AWS Shield

DDoS (Distributed Denial of Service) attacks are attempts to disrupt normal traffic by flooding a server with large amounts of fake traffic.

AWS Shield is a managed DDoS protection service that safeguards applications running on AWS. When you route traffic through Route53 or CloudFront, you are using AWS Shield Standard.  
Protects against attacks on layers:

- 3 \- Network layer
- 4 \- Transport layer
- 7 \- Application layer
  - Via integration with AWS Web Application Firewall (WAF)

TIers:

- Shield Standard
  - Free
  - Access to tools and best practices to build a DDoS-resilient architecture
  - Automatically available on all AWS services
- Shield Advanced
  - Costs (\$3000 / year)
  - Available on:
    - Amazon Route53
    - Amazon CloudFront
    - Elastic Load Balancing (ELBs)
    - AWS Global Accelerator
    - Elastic IP
      - Amazon EC1
      - Network Load Balancer
  - Notable features:
    - Visibility and reporting on layers 3, 4, & 7
    - Access to team and support
    - DDoS cost protection
    - Comes with SLA

## AWS Web Application Firewall (WAF)

Protects web applications from common web exploits. Write custom rules to allow or deny traffic based on the contents of HTTP requests. Otherwise, use a ruleset from a trusted AWS Security Partner from the AWS WAF Rules Marketplace.  
Can be attached to:

- CloudFront
- Application Load Balancers

Protect web apps from attacks covered in the OWASP top 10 most dangerous attacks:

1. Injection
2. Broken Authentication
3. Sensitive data exposure
4. XML external entities (XXE)
5. Broken access control
6. Security misconfigurations
7. Cross site scripting (XSS)
8. Insecure deserialisation
9. Using components with known vulnerabilities
10. Insufficient logging and monitoring

## Open API

OpenAPI specification (OAS) defines a standard, language-agnostic interface to RESTful APIs allowing humans and computers to discover and understand the capabilities of a service without having access to the source code, documentation or network traffic inspection.  
Swagger and OpenAPI used to be the same thing but as of OpenAPI v3, OpenAPI \= specification, Swagger \= tools to implement specification.  
OpenAPI can be JSON or YAML.  
AWS extends the OpenAPI definitions, allowing definition of AWS API Gateway-specific features in an OpenAPI file (e.g. policies, CORS).

## Amazon API Gateway

API Gateways are programs sitting between a single entry point and multiple backends, allowing for throttling, logging, routing logic or formatting of requests & responses.  
Amazon API Gateway allows the creation of secure APIs at any scale, acting as a front door for applications to access data, business logic or functionality from backend services.  
Can have caching or logs and lead to lambda functions, elastic containers, dbs etc.  
Versions (not incremental \- different use cases):

- REST API (v1)
  - Complete control over request and response
  - Most feature-rich
  - Higher costs
  - Public and private API options
- HTTP API (v2)
  - Low latency
  - Simpler feature set
  - Low costs
  - Only public APIs
- WebSockets API
  - Persistent connections for real-time use cases

For both REST and HTTP APIs you can import an Open API 3 file when creating the API.

### REST vs HTTP

Learn for exam.

#### Endpoint Types

| Type           | REST | HTTP |
| -------------- | ---- | ---- |
| Regional       | Yes  | Yes  |
| Edge-optimised | Yes  | No   |
| Private        | Yes  | No   |

#### Security

| Type                          | REST | HTTP |
| ----------------------------- | ---- | ---- |
| Mutual TLS authentication     | Yes  | Yes  |
| Certificates for backend auth | Yes  | No   |
| AWS WAF                       | Yes  | No   |

#### Authorisation

| Type                                 | REST                     | HTTP |
| ------------------------------------ | ------------------------ | ---- |
| Resource policies                    | Yes                      | No   |
| IAM                                  | Yes                      | Yes  |
| Amazon Cognito                       | Yes                      | Yes  |
| Custom auth with AWS Lambda function | Yes                      | Yes  |
| JWT                                  | No (possible via Lambda) | Yes  |

#### API Management

| Type                        | REST | HTTP |
| --------------------------- | ---- | ---- |
| Custom domains              | Yes  | No   |
| API keys                    | Yes  | Yes  |
| Per-client rate limiting    | Yes  | Yes  |
| Per-client usage throttling | Yes  | Yes  |

#### Development

| Type                    | REST | HTTP          |
| :---------------------- | :--- | :------------ |
| Automatic deploys       | No   | Yes (default) |
| User-controlled deploys | Yes  | Yes           |
| CORS config             | Yes  | Yes           |
| Test invocations        | Yes  | No            |
| Caching                 | Yes  | No            |
| Custom gateway response | Yes  | No            |
| Canary release deploys  | Yes  | No            |
| Request validation      | Yes  | No            |
| Request body transform  | Yes  | No            |
| Request param transform | Yes  | Yes           |

#### Monitoring

| Type                 | REST | HTTP |
| -------------------- | ---- | ---- |
| CloudWatch metrics   | Yes  | Yes  |
| CloudWatch logs      | Yes  | Yes  |
| Amazon Data Firehose | Yes  | No   |
| Execution logs       | Yes  | No   |
| AWS X-ray tracing    | Yes  | No   |

#### Integrations

| Type                                | REST | HTTP                   |
| ----------------------------------- | ---- | ---------------------- |
| Mock integrations                   | Yes  | No                     |
| Public HTTP endpoints               | Yes  | Yes                    |
| AWS services                        | Yes  | Yes (limited services) |
| AWS Lambda functions                | Yes  | Yes                    |
| Private integrations with NLB       | Yes  | Yes                    |
| Private integrations with ALB       | No   | Yes                    |
| Private integrations with Cloud Map | No   | Yes                    |

### REST

Components:

- Stages \- versions of the API
  - E.g. prod, staging, dev
  - Must be deployed to be accessible
- API \- container for multiple resources
- Resources \- represent endpoints
  - Resources can be nested within other resources
    - E.g. /users/show
- Methods \- individual methods for specific endpoints
  - E.g. GET, PUT
  - Customise requests and responses
  - Flow:

1. Method request
2. Integration request
3. Integration
4. Integration response
5. Method response

- Integration \- service that will be called
  - Lambda functions
  - HTTP, Mock
  - AWS Service
  - VPC link

### HTTP

Components:

- Stages \- versions of the API
  - E.g. prod, staging, dev, \$default
  - Auto-deployed to default
- API \- container for multiple routes
- Routes \- represent an endpoint
  - Can be nested within other routes
  - Choose one method and one endpoint
    - E.g. PUT /hello
    - One route for each method-endpoint combination you want to support
- Integration \- service that will be called
  - Lambda functions
  - HTTP, Mock
  - AWS service
    - Limited to a handful
      - EventBridge
      - SQS
      - AppConfig
      - Kinesis Data Streams
      - Step Functions
  - VPC link

## AWS Firewall Manager

Centrally configure and manage firewall rules across accounts and applications.  
Services that can be managed:

- AWS WAF (including classic)
- AWS Shield Advanced
- Security Groups
- Network Access Controls
- AWS Network Firewall
- Amazon Route53 Resolver DNS Firewall
- Third-party firewall services

Requirements:

- Account must be member of AWS Organisation
- Account must be AWS Firewall Manager admin
- Must have AWS Config enabled for accounts and regions
- Must have Resource Access Manager (RAM) enabled for specific services
  - E.g. AWS Network Firewall, Route53 resolver DNS firewall

Policy config varies based on targeted service.  
\$100 per month. Many policies can be achieved via AWS Config (often used under the hood with these types of services).

### GraphQL

Open-source agnostic query adaptor allowing querying of data from many different data sources. Used to build APIs where clients will send a query for nested data. Mitigated the issue of versioned or rapidly-changing APIs compared to REST because just the wanted data can be requested.  
GraphQL schemas (written in GraphQL Schema Definition Language) composed of:

- Types \- represent objects and their fields
- Fields
- Queries \- define the exact shape of the data needed by the client
- Mutations \- allow for data to be created, updated or deleted
- Subscriptions \- support live updates sent from the server to the client

### AWS AppSync

Fully-managed GraphQL service. Uses Resolvers attached to specific fields within types in the schema to implement state-changing operations for the query, mutation and subscription field operations. Can support real-time APIs via GraphQL subscriptions feature.  
API types:

- GraphQL APIs \- single API from multiple data sources
- Merged APIs \- collection of Graph APIs that act as one

Data sources:

- DynamoDB tables
- Amazon OpenSearch
- AWS Lambda function
- HTTP endpoint
- Amazon EventBridge
- RDS

Caching options:

- None \- resolvers will always fetch from data sources
- Full request caching \- cache all requests
- Per-resolver caching \- specific operation or field defined in a resolver will return responses from the cache

Caching requires the provision of an on-demand instance \- not serverless.  
Authorisation types:

- API key
- AWS IAM
- Amazon Cognito User Pools

AppSync supports custom domains and has a query editor built into the UI.
