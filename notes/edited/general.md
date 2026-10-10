# General

# AWS API

Management Console, CLI, SDKs and plain HTTP requests all access resources via the AWS API (Application Programming Interface).

## CLI

**Terminal** \= text only interface  
**Console** \= physical computer to input information into a terminal  
**Shell** \= command line program user interact with to input commands

AWS **CLI** is an executable program. Current version (v2) bundles its own Python; v1 required Python and is in maintenance mode  
Can be installed on Windows, Mac or Linux.

Order of priority for variables \= CLI **parameters** \> **env** vars \< **config** files.

## Keys

**Access keys** are generated **per user** (can have **two** active keys). These keys are used to authenticate and have the same access as the user. Two methods of using keys: storing in **\~/.aws/credentials** (TOML format), which can provide keys for multiple profiles, and **exporting** as env vars.

## Retries

Common to get networking issues when interacting with any API over a network. Standard is to **retry** with **exponential backoff** (e.g. retry in 1 second, retry in 2 seconds, retry in 4 seconds etc.). Often this is built into SDKs or CLIs, with options.

## STS

Security Token Service (**STS**) is a way of getting temporary, limited-privilege credentials for IAM **users** or federated users. Default endpoint is global (legacy) **https\://sts.amazonaws.com**; Regional endpoints are also available and recommended (lower latency, redundancy). Will return: **AccessKeyID, SecretAccessKey, SessionToken** & **Expiration**. Could use instead of long term access keys for more security by having user with permissions to assume roles but not access directly and use STS every time.

## Signing

Requests to AWS usually need to be **signed** to prevent data tampering and verify requester identity (not for anonymous access to S3 or select other requests). Requests using an SDK or CLI are signed automatically. **2 protocols** for signing: Signature **v2** (legacy) and Signature **v4**.  
In signature v4, select request elements are concatenated into a **‘string to sign’**, a **signing key** is created from the **secret access key** , a **signature** is created from a hash-based message authentication code (**HMAC**) of the **string to sign** using the **signing key**.  
The process to sign depends on the request type: usually in authorisation header, but can be in a post request or query parameters.

## IP Address Ranges

AWS publishes all IP ranges in JSON at **https\://ip-ranges.amazonaws.com/ip-ranges.json**. This can be useful for whitelisting IP addresses for example.

## Service Endpoints

To connect programmatically to an AWS service, you use the URL of the entry point to the service (**endpoint**) in the format: **protocol://service-code.region-code.amazonaws.com**. **Four** types: **Global**, **Regional** (region must be specified), **FIPS** (enterprise use, uses encryption), **Dualstack**. These types can be combined. A service can have **multiple** endpoints. SDKs and the CLI automatically use the default endpoint for a service in a region.

# Identity and Access Management (IAM)

IAM **creates** and **manages** users and groups, using **permissions** to allow and deny access to AWS resources.  
Terms:

- **Policies** \- JSON docs granting permissions for a specific **user**, **group** or **role** to access services. Attached to IAM **identities**.
  - Types:
    - Managed \- Managed by AWS, cannot edit. Have orange box next to name.
    - Customer Managed \- Customer created and editable.
    - Inline \- Policy attached directly to user, not reusable.
  - Made up of:
    - Version \- policy language version, not version of policy
    - Statement \- container for policy element (can have multiple), contains elements below
    - Sid (optional) \- label for statement
    - Effect \- whether policy will Allow or Deny
    - Action \- list of actions to allow / deny
    - Principal \- account, user, role or federated user to apply to (resource-based and trust policies only)
    - Resource \- relevant resource
    - Condition (optional) \- circumstances for permission activation
- **Permission** \- API actions that can or cannot be performed. Represented in IAM Policy document
- **Identities**:
  - **Users** \- End users logging into the console or interacting with AWS resources programmatically or via the UI.
  - **Groups** \- Sets of multiple users so they share permissions levels (e.g. Admins, Devs, Auditors)
  - **Roles** \- Grant AWS resource permissions to specific API actions. Policies associated with a role and then assigned to resource

## Principle of Least Privilege (PoLP)

Computer security concept of providing a user, role or application the **lowest** level of permission possible to achieve an operation or action.  
Contributing concepts:

- Just-Enough-Access (**JEA**) \- only providing permissions for the exact necessary **actions** to perform the required task
- Just-In-Time (**JIT**) \- only providing permissions for the **smallest duration** necessary to perform the required task
- Risk-based adaptive policies \- Any attempt to access a resource generates a **risk score** based on device, IP address, time and much more
  - AWS does **not** have any built in risk-based adaptive policies

## AWS Account Root User

AWS **Account** \= the account holding all the AWS resources  
AWS Account **User** \= user for common tasks assigned permissions  
AWS Account **Root** User \= special account with **full** access

- **Cannot** be deleted
- Uses email and password to log in
- **Full** permissions to account, **cannot** be limited (except by organizations service control policy (SCP)
- **One** root per **account**
- Recommended never use root user access keys
- Recommended MFA for root user

Use root user to:

- **Change account settings**
  - Email address, root user password, root user access keys (standalone accounts)
- Restore IAM user permissions
- Activate IAM access to Billing and Cost Management console
- View tax invoices
- **Close AWS account** (standalone accounts; member accounts via Organizations)
- **Change or cancel AWS Support plan**
- Register as Reserved Instance Marketplace seller
- Enable MFA delete on S3 buckets
- Edit or delete S3 bucket policy including invalid VPC or VPC endpoint IDs
- Sign up for GovCloud
- Create organization

In IAM you can set **Password Policies** to enforce **minimum requirements** and **rotation** of passwords.

## Temporary Security Credentials

Just like access keys but only last from **minutes** up to **several hours** (role sessions: default 1 hour, up to 12 hours). Not stored with the user but generated dynamically and provided when requested. Used under the hood for roles and identity federation.  
Used for:

- **Identity federation**
  - Linking electronic identity across identity management systems (e.g. logging in via Google account)
  - Enterprise identity federation (SAML, custom federation brokers)
  - Web identity federations (Amazon, Facebook, Google, OpenID Connect \[OICD\])
- **Delegation**
- **Cross-account** access
  - Allow users from other AWS accounts to assume roles in your account with access to AWS resources
  - STS **AssumeRole**
  - STS **AssumeRoleWithWebIdentity**
    1. Use OAuth to log into third party identity service
    2. Get JWT token back on success
    3. Call STS AssumeRoleWithWebIdentity passing JWT token
    4. Get back temporary credentials on success
    5. Use these to access resources using cross-account role
- **IAM roles**

## AWS Single-Sign on (SSO)

Create or connect workforce identities in AWS once and manage access central across organization. Managed user permissions centrally for AWS accounts, AWS applications and SAML applications.  
Identity sources:

- AWS SSO
- Active Directory
- SAML

### AWS Service Catalog

Create and manage catalogs of products approved for use on AWS. Alternative to granting direct access to AWS resources via the AWS console. Provides:

- Standardisation
- Self-service discovery and launch
- Fine-grained access control
- Entensibility and version control

Administrative side:

- Administrative User \- manages the catalog
- Portfolio \- container for sets of:
  - Product \- CloudFormation template
    - Can associate budget
    - Product must be removed from portfolio and not provisioned, to delete
  - Permissions \- who can view / launch products
    - Groups
    - Roles
    - Users
  - Constraints \- rules applied on provisioned products
    - Launch \- use IAM role instead of user credentials
    - Notifications \- send product notifications to a stack
    - Template \- limit the options when launching a product (e.g. limit hardware that can be provisioned to t2.micro)
    - StackSet \- configure product deployment across accounts and regions
    - TagUpdate \- allow or disallow end users updating tags on associated resources

Consumer side:

- End user \- uses the catalog
- Catalog \- user-friendly console to view / launch products
  - Products \- as above, viewable and launchable
  - Provisioned products \- active CloudFormation stacks
    - Service actions \- SSM documents to perform tasks on the stack

## Amazon Guard Duty

Threat detection system acting as both an intrusion detection system (IDS) and intrusion protection system (IPS). It continuously scans for malicious or suspicious activity using machine learning to analyse:

- CloudTrail logs
- VPC Flow logs
- DNS logs

It will alert you to findings and can automate an incident response via CloudWatch Eventbridge or third-party services. You can investigate an issue with Amazon Detective.

## AWS Health Dashboard (formerly Personal Health Dashboard)

Provides alerts and guidance for AWS events that might affect your environment. Shows recent events to help you manage active events and shows proactive notifications enabling planning for scheduled activities. Use alerts to be notified about changes that can affect AWS resources and follow guidance to resolve issues.

Not to be confused with the public Service health page, which shows general health of AWS services divided by high level region (North America, Europe, etc.) and needs no sign-in. An icon and details column indicate the status of each service.

## AWS Artifact

Self-service portal for on-demand access to AWS compliance reports. Choose compliance type and download pdf. Explains AWS responsibilities and customer responsibilities to ensure standard is being met.

## ML Services

### Amazon CodeGuru

Machine-learning code analysis service comprised of:

- CodeGuru Security \- detect, track and fix code security issues
  - Code security analytics scan
  - Code quality analytics scan
  - Secrets detection scan
- CodeGuru Profiler \- find and fix inefficiencies in code
- CodeGuru Reviewer \- associate a repo and get continuous code change recommendations

Supports various languages.

### Amazon Comprehend

Natural Language Processor (NLP) service to find relationships between text in order to produce insights \- e.g. looking at data like customer emails, support tickets etc.  
Can analyse text and extract:

- Entities \- (e.g. person, organisation, location)
- Key phrases \- text appearing important (e.g. You need to **pay** the amount of **\$200** by **September 31st**.)
- Language \- (e.g. confidence of the language being spoken)
- PII
- Sentiment \- attitude towards the text (e.g. 0.20 negative)
- Targeted sentiment \- specific words and their attitude (e.g. awful 1.0 negative)
- Syntax \- identify parts of a language (e.g. hello proper noun)
- Custom models \- upload training data to analyse and extra custom text
  - Amazon Comprehend Flywheel \- automates the training of model versions for custom models

Comprehend is serverless and billed on size of requests in units e.g. 1 unit \= 100 characters. Real-time analysis can be performed via an endpoint (or custom endpoint for custom models). Analysis jobs allow for batch jobs. Provides confidence scores for each finding.

### Amazon Forecast

Time-series forecasting service for business outcomes such as product demand, resource needs or financial performance. Need to upload dataset to S3 with historical data and optional additional metadata.  
Workflow:

- Create data set group / data import job
  - Define schema
  - Register task
- Create predictor / get accurate metrics
  - ELT job evaluates the model
    - Choose predefined backtest
- Create forecast
  - Deploy the predictor
  - Retrained with full dataset
- Query / export forecast

Produced visual graph.

### Amazon Fraud Detector

Fully-managed fraud detection service identifying potentially fraudulent online activities e.g. online payment fraud, creation of fake accounts. Comes with predefined models against which you train your data:

- Online fraud insights \- optimised for fraud with little historical data (new account registration)
- Transaction fraud insights \- test for fraud cases where entity being evaluated might have history the model can use to improve prediction accuracy
- Account takeover insights \- for accounts compromised by phishing or other attack

Often used with step functions, Lambda, Kinesis etc for real-time fraud detection and alerts.  
Upload training dataset to S3.  
Components:

- Models \- (user defined)
- Thresholds \- (user defined)
- Scores \- numerical values representing estimated risk level (model generated)
- Rules \- interpret variable values during fraud prediction (user defined)
- Detector
- Outcomes \- define the fraud prediction result e.g. risk level and actions (user defined)
- Events
  - Entities \- who is performing the event (e.g. customer)
  - Labels \- classify an event as fraudulent or legitimate
  - Variables \- data points used in model (e.g. location, transaction amount)

### Amazon Kendra

Enterprise machine learning search engine service using natural language to suggest answers to questions instead of basic keyword matching. ‘Like interacting with a human’. Amazon Lex chatbot can be used as an interface to Kendra. Returns document results \- more like a librarian recommending books based on your question rather than them having knowledge to answer your question about any book..  
Components:

- Index \- table holding indexes of your documents to make them searchable
- Data source \- where documents are stored (e.g. S3, Sharepoint, Postgres)
  - Data source template schemas \- AWS provides around 40 schema templates for common AWS services or third-party cloud storage services
- Document addition API \- API to add documents directly to an index

Versions:

- Developer
  - 5 indexes with 5 data sources each
  - 10,000 documents / 3GB extracted text
  - 4,000 queries per day / 0.05 per second
  - 1 AZ
- Enterprise
  - 5 indexes with 50 data sources each
  - 10,000 documents / 3GB extracted text
  - 8,000 queries per day / 0.1 queries per second
  - 3 AZs

### Amazon Lex

Conversation interface service to build voice and text chatbots. Provides natural language understanding (NLU) and automatic speech recognition (ASR).  
Use:

- AWS-provided bot templates for common industries
- Transcripts to create a new bot
- Gen AI to build a bot by describing it
- Target languages and AWS-provided voices

Integrates with AWS Lambda to connect with other services.  
Amazon Lex Network of Bots is a feature to add multiple bots to a network and intelligently route the query to the appropriate bot.  
Components:

- Bot \- performs automated tasks, input to interact with conversational model
  - Version \- snapshot of bot model
    - Alias \- label/tag for version
  - Language \- target language(s) bot can converse in
- Intent \- action the user wants to perform
  - Sample Utterance \- text describing intent from user POV
    - “Can I order a pizza?”
    - “I want pizza”
    - “Hey there, any chance of a pizza?”
  - Slot \- inputs an intent requires from the user (can be 0\)
    - Slot type \- data type of the slot
      - Custom enumeration values (e.g. “Small”, “Medium”, “Large”)
      - Predefined data type (e.g. AMAZON.Number)

### Amazon Personalize

Real-time recommendations service. Used to make product recommendations to customers shopping on Amazon.  
Process:

1. Create **data set group**
2. Upload **data set** to data set group (**CSV** files)
   1. Provide multiple data sets
      1. User item interaction data (**required**)

- Core dataset used to train custom model
- Contains at least:
  - USER_ID
  - ITEM_ID
  - TIMESTAMP  
    2. User data (optional)
- Metadata about users to improve recommendation quality
- Contains at least USER_ID  
  3. Item data (optional)
- Metadata about the items to improve recommendation quality
- Contains at least:
  - ITEM_ID
  - CATEGORY_L1
  2. Need a **JSON schema mapping** for CSV files
  3. Reference the dataset location from an S3 object location

3. **Solutions** and **Recipes** allow fine tuning of models
   1. **Solutions** helps generate recommendations
   2. **Recipes** are predefined AWS algorithms
4. **Event trackers** \- Can track user events and feed into solution using Ingestion SDK
5. **Filters** \- Remove certain items from recommendations based on rules
6. Launch a **Campaign** to allow applications to get recommendations from solutions

### Amazon Polly

Text-to-speech service providing a spoken audio file in a synthesised voice in response to uploaded text.  
Engine types:

- Standard (\$) \- not as natural-sounding as other engines, but most cost-effective
- Long form (\$\$) \- sounds more natural than standard when reading longer text
- Neural (\$\$\$) \- most natural-sounding speech, supports newscaster and narration speaking styles

No standard speed \- variation between voices.  
Lexicon \- for specialised pronunciation

- Lexicon file (.xml, .pls) with up to 40,000 chars and 100 pronunciation rules

Speech Marks \- metadata describing the speech

- Where a word starts / ends
- Utilise Speech Synthesis Markup Language (SSML) (XLM-based)
- Integrate with Visme (third-party marketing material creation service)

### Amazon Rekognition

Image and video recognition service.  
Prebuilt use cases:

- Object detection
- Face detection
- Searching faces in connection
- People pathing
- Detecting personal protective equipment
- Recognising celebrities
- Moderating content
- Detecting text
- Detecting video segments
- Detecting face liveness

Also allows custom labels to identify specific objects, logos etc. specific to business needs.  
Image requirements:

- Jpg / png format
- Base64 encoding (automatically done by SDKs)

Can access images from S3 bucket.

Amazon Textract  
Service to extract text from scanned documents (OCR) to digitally extract data from paper forms.  
Capabilities:

- OCR documents
  - Retain layout coordinates
  - Convert to a table
  - Detect forms
  - Query against the OCR data
  - Detect signatures
- OCR expenses (e.g. receipts)
- Analyse IDs (e.g. drivers license / passport)
- Analyse lending (mortgage documents)
- Custom queries \- train your own models with uploaded samples

Amazon Translate  
Neural machine learning text translation service.  
Processing modes:

- Real-time translation
- Async batch processing

Provide:

- Text
- Source language
- Target language

### AWS Auto Scaling

Service centralising auto scaling resources. Can manage and make recommendations for:

- EC2 Auto Scaling Groups
- ECS EC2 (uses auto scaling groups under the hood)
- Amazon Aurora
- Amazon DynamoDB
- Spot Fleet

Easily apply Dynamic or Predictive scaling.

### AWS Amplify

Opinionated framework and fully-managed infrastructure to allow developers to focus on building web and mobile applications.  
Encompasses:

- Amplify CLI \- unified toolchain to create, manage and integrate AWS cloud services
- Amplify SDK \- connects AWS services to client-side code
- Amplify UI \- accessible, themeable, performant React components directly connected to cloud
- Amplify Hosting \- static website hosting platform
- Amplify Studio \- visual dev environment for building full stack web and mobile apps

Has direct integrations with:

- Amazon Cognito \- for auth signin and signup (must use amplify for this now)
- API Gateway \- for rest APIs
- AppSync \- for GraphQL APIs
- S3 \- for storage or static website hosting
- DynamoDB \- for backend db
- Lambda \- for custom resolvers

Works with js frameworks:

- JavaScript
- React
- Flutter
- Swift
- Android
- React Native
- Angular
- Next.js
- Vue

There is a gen 2 now.

### AWS Batch

Plans, schedules and executes batch computing workloads across the full range of AWS compute services. Can utilise Spot Instance to save money.  
Definitions:

- Job \- named unit of work (e.g. Shell script, docker container image)
- Job Definition \- defines how to run the job (e.g. amount of compute / memory)
- Job Queue \- collection of jobs that determine job priority
- Job Scheduler \- evaluates when, where and how to run jobs that are submitted to a queue (defaults to FIFO)
- Job Dependencies \- allow specifying a job ID to another job to wait for it before scheduling processing

Batch can run jobs on:

- EC2
- Fargate
- EKS

Job types:

- Array job \- shares common parameters (e.g. job definition, vCPUs, memory)
- Multi-node parallel job \- run single jobs that span multiple EC2 instances
- GPU job \- run on EC2 GPU-based instance types

### AWS Device Farm

Application testing service in different real environments \- not simulators.  
Supports:

- Mobile device testing
  - Native iOS / Android / mobile web app
  - Built-in tests Fuzz can randomly test actions
  - Videos captured of runs
  - Various real devices available
  - Can test using Appium suite
- Desktop browser testing
  - Various web browsers available
  - Selenium to write tests

### AWS Elastic Transcoder

Fully-managed video transcoding service, converting from one format to another for on-demand video (VOD) or streaming. Cannot be used via CloudFormation (AWS SDK / AWS CLI for automation only).  
To create a pipeline job, choose a preset (what to convert the video to) and a source and destination bucket.  
New version of Elastic Transcoder is called AWS Elemental MediaConvert. Elastic Transcoder was discontinued 13 Nov 2025.

### AWS Elemental Media Convert

Fully-managed video transcoding service \- new more comprehensive version of Elastic Transcoder with more processing options:

- Video correction
- Input filtering
- Cropping
- Overlaying
- Audio track / caption insert
- Etc.

Define job, input and outputs. Pulls from source bucket, transcodes and places in destination bucket.

## AWS Lambda

Serverless functions as services, allowing code to run without provisioning or managing servers. Costs for compute time consumed \- no cost when not running. Lambda executes only when needed and scales automatically up to the account concurrency limit (default 1,000 per Region, can be raised). Supports various runtimes including custom environments. Commonly used to glue services together without having constantly running servers. Lambda can be triggered by many services including external.  
Invocation types:

- Sync \- wait for a response (e.g. HTTP response)
- Async \- immediately returns to async AWS services such as:
  - SNS topic
  - SQS queue
  - Lambda function
  - EventBridge bus

Can configure:

- Timeout \- default=3s, min=1s, max=15m
- Storage \- default=512MB, min=512MB, max=10GB

Function Versions  
Can manage deployment of Lambda functions (e.g. \$LATEST). Each version has its own ARN. The ARN of the function without the version suffix is the unqualified version (e.g. arn:aws:lambda:aws-region:acc-id:function:helloworld); the ARN with the version suffix is the qualified version (e.g. arn:aws:lambda:aws-region:acc-id:function:helloworld:\$LATEST). These two are effectively identical because unqualified ARNs point to the latest version. Unqualified ARNs cannot use Aliases \- friendlier names to specific versions when referring to them programmatically.

Layers  
ZIP archives containing libraries, custom runtimes or other dependencies, allowing them to be used without including in the deployment package.  
Limits:

- Up to 5 layers per function
- Total unzipped deployment package up to 250MB

Instruction Sets  
Two available instruction architectures for lambda: arm64 and x86_64. arm64 (Graviton2) typically gives better price-performance. All AL2 runtimes support both architectures.

Lambda Runtimes  
Preconfigured environments to run specific programming languages. Useful because they don’t require you to configure a container or OS config. Fully-managed and secure. Older runtimes are deprecated and upgrades are required.

OS-Only Runtimes  
Used when there is no preinstalled programming language / specific libraries installed when you want to compile.  
Cases:

- Native ahead-of-time (AOT) compilation
  - Languages like Go, Rust, C++ compile natively to an executable binary which doesn’t require a dedicated language runtime
  - Must include a runtime interface client in the binary
  - Must compile binary for Linux env for same instruction set architecture
- Third-party runtimes
  - Community or company-created runtimes for languages not supported normally
- Custom runtimes
  - Build your own

OS-only runtimes: Amazon Linux 2 \= provided.al2, Amazon Linux 2023 \= provided.al2023.

Deployment Packages  
Package containing the function code Lambda will deploy.  
Types:

- ZIP archive \- zip contents of code and additional libraries
  - AWS uploads contents to AWS-managed S3 bucket or you can upload to S2 bucket and reference address in Lambda
    - ZIP \> 50MB must be uploaded second way
  - Rely on Lambda runtimes (limited to these envs)
- Container image \- build a container
  - Create Dockerfile
  - Build image
  - Push image to registry (e.g. Amazon Elastic Container Registry \- ECR)
  - Reference container image to Lambda
  - Do not need to specify runtime (you’re doing it in the image) but slower

## Step Functions

Coordinate multiple AWS services into serverless workflows by creating a state machine (abstract model moving between states based on conditions) with micro-services. Displayed in visual workflow. Automatically triggers, tracks and logs each step and retries when there are errors.  
State machine types:

- Standard \- general purpose (long workloads)
- Express \- for streaming data (short workloads)

Steps can be executed in parallel. States are configured through Amazon States Language (JSON).

### Use cases:

- Manage a Batch job or Fargate container
  - Run, then notify SNS on success or failure
- Transfer data records
  - Load DynamoDB
  - Add each item to SQS queue
  - Remove items from DB as they are added to queue
  - Report success when done

\[Read through use cases / examples in documentation for exam\]

### States

**Pass states** pass their input to output without performing real work (mocks). Useful when constructing / debugging state machines.  
Made up of:

- Parameters \- key-value pairs passed as input
  - E.g. “ship”: “enterprise”
- Result \- virtual task to be passed to next state
  - E.g. “government”: “federation:
- ResultPath \- where to place the output of the mock
  - E.g. “\$.politics”
- Output \- what it produces
  - E.g. { “ship”: “enterprise”, “politics”: { “government”: “federation”}}

**Task states** are units of work performed by a state machine. Work performed by:

- Calling an AWS Lambda function
  - Pass the lambda ARN
- Passing parameters to API actions of other services
  - Pass ARN & parameters (varying per service)
  - Available services:
    - Lambda
    - AWS Batch
    - DynamoDB
    - ECS/Fargate
    - SNS
    - SQS
    - SageMaker
    - EMR
    - StepFunctions
- Using an activity
  - Workflow waits for activity worker to poll for the task
  - Allows task to be completed by a worker hosted anywhere

**Choice states** add branching logic to a state machine. Use logical conditions with NOT, AND operators etc to construct branches. Default branch can be given.

**Wait states** delay the state machine from continuing for a specified time. Can wait defined time, or wait until.

**Succeed states** stop execution successfully. Useful for branches that do nothing but stop execution.

**Failure states** stop the execution of the state machine while marking as a failure. They can provide types, causes and errors.

**Parallel states** execute steps in parallel and do not advance the state machine until they are all complete.

**Map states** iterate over an array, completing the step for each item. Good for inserting records to database etc.

### Inputs & Outputs

Step functions take JSON event data as input and produce JSON as output. The JSON payload can be manipulated using:

- InputPath \- select a portion of the state input
- Parameters \- create a collection of key-value pairs that are passed as input
  - When using JSONPath for parameters, append “.\$” to the key name
- ResultsSelector \- change a state’s result before the ResultPath is applied (opposite to parameters essentially)
- ResultPath \- determines what should be output \- the input, task output or a combination
- OutputPath \- select a portion of the state output to pass to the next state

Amazon States Language uses JSONPath syntax to identify components. This is a query language for JSON using an expression starting with the root (\$) and using dot or bracket notation to select children.

## Key Management Service (KMS)

Makes it easy to create, control and rotate encryption keys on AWS.  
Integrates with many services:

- RDS
- CodeCommit
- S3
- CodeDeploy
- Glacier
- SNS
- SQS
- DynamoDB
- EC2
- X-Ray
- ElastiCache
- Codebuild
- CloudTrail
  - To audit access history
- Many more

Most AWS services can just checkbox encryption and choose a KMS key \- simple.

- Managed ones free
- Custom ones have small ongoing cost

KMS is a multi-tenant Hardware Security Module (HSM).

- HSM \= hardware specialised for storing encryption keys
  - Designed to be tamper-proof
  - Stores keys in memory, so not written to disk
- Multi-tenant \= multiple customers using same hardware
  - Isolated virtually
  - CloudHSM is single-tenant HSM for stricter compliance
    - FIPS 140-2 level 3 complaint, vs level 2 for multi-tenant

CLI commands to know:

- aws kms …
  - create-key \- creates a unique customer managed customer master key in AWS account and region
  - encrypt \- encrypts plaintext into ciphertext using a customer master key
  - decrypt \- descrypts ciphertext that was encrypted by an AWS KMS CMK into plaintext
  - re-encrypt \- decrypts ciphertext and then re-encrypts it within KMS. Why?
    - Manually rotate a CMK
    - Change the CML that protects ciphertext
    - Change the encryption context of ciphertext
  - enable-key-rotation \- enables automatic rotation of the key material for the specified symmetric CMK
    - Must be symmetric
    - Cannot be performed on a CMK in a different account

### Customer Master Key (CMK)

Keys used to encrypt all other keys (data keys) on a system (envelope encryption). Master keys are stored in secure hardware. Customer master keys are logical representations of a master key.  
Includes:

- Encryption key material
- Decryption key material
- Metadata:
  - Key ID
  - Creation date
  - Description
  - Key state

KMS supports symmetric and asymmetric CMKs.

## CloudHSM

Single-tenant HSM as a service, automating hardware provisioning, software patching and backups. FIPS 140-2 level 3 compliant as single-tenant. Built on Open HSM industry standards to integrate with:

- PKCS\#11
- Java Cryptography Extensions (JCE)
- Microsoft CryptoNG (CNG) libraries

Can transfer keys to other HSM solutions, easy to migrate keys. Can configure AWS KMS to use CloudHSM cluster as a custom key store rather than using the default.  
More expensive than KMS.

## Amazon Certificate Manager (ACM)

Provision, manage and deploy public and private SSL/TLS certificates for use with AWS services.  
Certificate types:

- Public \- certificates provided by ACM
  - Free
- Private \- imported certificates
  - Costs (\$400/month\!)

Can be attached to:

- ELB (including ALB)
- CloudFront
- API Gateway
- Elastic Beanstalk (via ELB)

ACM can handle multiple subdomains and wildcard domains (e.g. example.com &\*.example.com).  
Example implementation:

- Traffic ingress from the internet, via Route53, through an ALB and into EC2 instances
- ACM certify at the ALB to terminate SSL at the load balancer
- This prevents you needing to install certificates on every instance
- Theoretically less secure, but unencrypted traffic is on own network at least

## Amazon Cognito

Customer Identity and Access Management (CIAM) system, providing authentication, authorisation and user management for web and mobile apps. Provides authentication to AWS services.  
Methods:

- Cognito User Pools
  - User directory with authentication to identity providers (IdP) to grant access to an app
- Cognito Identity Pools
  - Provide temporary credentials for users to access AWS services
- Cognito Sync
  - Syncs user data and preferences across all devices

Amazon Detective  
Analyse, investigate and quickly identify the root causes of security findings or suspicious activities.  
You can:

- Quickly identify trends for EC2 instances and IAM Principals for example
- See on a map where API calls are generally being made from
- See a summary list of how many times specific API calls have been made
- See how much groups of API calls have increased in volume
- Launch investigations on specific IAM principals to see if they are utilising specific tactics
  - Amazon Detective creates a behaviour graph when making determinations

## Amazon Directory Service

Provides multiple ways to use Microsoft Active Directory (AD). It lets you use Microsoft AD-aware or Lightweight Directory Access Protocol (LDAP) \-aware apps in the cloud.  
Offers:

- Simple AD (not in all regions) \- AD-compatible directory supporting very basic features
- AD Connector \- proxy service to connect to existing on-premise AD
- AWS Managed Microsoft AD \- full feature version managed by AWS

### Directory service

Maps names of network resources to their network addresses. Shared information infrastructure for locating, managing, administering and organising resources like:

- Volumes
- Folders
- Files
- Printers
- Users
- Groups
- Devices
- Telephone numbers
- Other objects

Critical component of a network operating system.  
Directory server (name server) \- server providing a directory service.  
Each resource on the network is considered an object and information about it is stored as a collection of attributes.  
Well-known directory services include:

- Domain name service (DNS)
- Microsoft Active Directory
  - Introduced in Windows 2000
  - Manage multiple on-premise infrastructure components and systems using a single identity per user
  - Forest of domain trees, each of which representing an organisational unit
- Etc.

### Lightweight Directory Access Protocol (LDAP)

Open, vendor-neutral industry standard application protocol for accessing and maintaining distributed directory information services over an IP network. Commonly used to provide a central place to store usernames and passwords. LDAP allows for same-sign on \- allowing users to have a single ID and password to access everything, which they need to enter each time.  
Example setup:

- On-premise active directory
- Communicates directly with an LDAP directory
- Which communicates with Google Cloud, Kubernetes and Jenkins

LDAP vs. SSO:

- Most SSO uses LDAP under the hood
- LDAP was not designed for web-apps
- Some systems only support integration with LDAP \- not SSO

## Secrets Manager

Used to safely store and rotate secrets:

- Database credentials:
  - RDS
  - Redshift
  - DocumentDB
  - Other databases
- Key/Value:
  - Usually API keys

Enforces encryption at rest using KMS.  
Costs per secret per month and per 10,000 API calls.  
CloudTrail can monitor credentials access in case of audit.  
Database credentials can be automatically rotated:

- Intervals range from 30 days \- 365 days
- Performed by a Lambda function
- Can rotate password for the superuser or developer programmatically accessing the db

## AI Dev Tools

### Amazon Q

AI chatbot using multiple LLM models via Amazon Bedrock. Ask questions similar to ChatGPT or other generative AI chat services.  
Models:

- Amazon Q Business
  - Connect it to company data, information and systems
  - More than 40 built-in connectors
- Amazon Q Developer
  - Coding, testing and upgrading to troubleshooting and optimising AWS resources
  - Integrated into:
    - AWS Managed Console
    - VSCode via AWS toolkit
    - Cloud9
    - AWS Lambda Code Editor
    - Slack
    - More
- Amazon Q for Amazon QuickSight
  - Ask questions about BI data within QuickSight
  - Generative BI capabilities to:
    - Quickly build compelling visuals
    - Summarise insights
    - Answer data questions
    - Build data stories using natural language
- Amazon Q for Amazon Connect
  - Real-time conversation with the customer along with relevant company content
- Amazon Q for AWS Supply Chain
  - Get intelligent answers about what is happening in a supply chain

### Amazon CodeWhisperer

Real-time coding companion \- suggesting code as you’re writing.  
Provides:

- In-line code suggestions
- Public code filter and reference tracking
- Command line integration
- Amazon Q chat in IDE
- Security vulnerability scanning

Tiers:

- Individual
  - Authenticate via Builder ID
  - 50 users / month
- Professional
  - Authenticate via IAM Identity Centre
  - 500 users / month
  - Additional features:
    - Customise for organisations
    - Organisational license management
    - Organisational policy management
    - Amazon Q feature development
    - Amazon Q code transformation

Integrates with:

- AWS Glue Studio Notebook
- JetBrains (e.g. IntelliJ IDEA)
- JupyterLabe
- Amazon SageMaker Studio
- Terminal, shell & command-line
- VSCode
- Visual Studio

In the AWS Toolkit.
