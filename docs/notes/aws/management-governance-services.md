---
description: AWS exam notes on management and governance, covering CloudFormation, CDK, CloudWatch, X-Ray, Health Dashboard, Managed Prometheus and Grafana, Trusted Advisor, Compute Optimizer, Organizations, Control Tower and RAM.
tags:
  - AWS
  - Management & Governance
  - Exam Prep
---

# AWS Management & Governance Services

Quick revision notes for the Solutions Architect exam. These services help you **provision** infrastructure as code, **monitor** what's running, **optimise** cost and performance, and **govern** many accounts. The table below gives one line per service; the sections after it go deeper on the ones that show up most in questions.

## At a glance

| Service | Category | One-liner |
|---|---|---|
| **CloudFormation** | Provisioning (IaC) | Declare AWS resources in JSON/YAML templates and deploy them as repeatable stacks |
| **CDK** | Provisioning (IaC) | Define infrastructure in a programming language (TypeScript, Python…); synthesises to CloudFormation |
| **CloudWatch** | Monitoring | Metrics, logs, alarms and dashboards for AWS resources and apps in real time |
| **X-Ray** | Monitoring | Distributed tracing to follow a request across services and find bottlenecks |
| **Health Dashboard** | Monitoring | AWS service events, plus events and scheduled changes that affect *your* account |
| **Managed Prometheus & Grafana** | Monitoring | Managed open-source Prometheus (metrics) and Grafana (dashboards) |
| **Trusted Advisor** | Optimisation | Checks your account against best practices for cost, security, performance and limits |
| **Compute Optimizer** | Optimisation | ML recommendations to right-size over- or under-provisioned compute |
| **Launch Wizard** | Provisioning | Guided, best-practice deployments of third-party apps like SQL Server, SAP and Active Directory |
| **Organizations** | Governance | Group many AWS accounts into a hierarchy with central policies (SCPs) and consolidated billing |
| **Control Tower** | Governance | Sets up and governs a secure multi-account landing zone, built on Organizations, with guardrails |
| **Resilience Hub** | Resilience | Assesses an application's resilience against your RTO/RPO targets and recommends improvements |
| **Resource Explorer** | Inventory | Search for and find resources across Regions and accounts |
| **Resource Access Manager (RAM)** | Sharing | Share resources (subnets, Transit Gateways…) with other accounts without duplicating them |

---

## Infrastructure as code

### AWS CloudFormation

The **blueprint** for your infrastructure: describe resources in a **template** (JSON or YAML) and CloudFormation creates, updates and deletes them together as a **stack**.

**Benefits**: infrastructure as code, **repeatable** deployments across environments and Regions, **version control** of templates, **resource tracking** (everything in a stack is tagged and managed together), and saves time and cost (delete a whole environment in one step).

**Template sections**

| Section | Purpose |
|---|---|
| **Resources** (required) | The AWS resources to create |
| **Parameters** | Inputs at deploy time (instance type, environment name) |
| **Mappings** | Static lookup tables (e.g. AMI ID per Region) |
| **Conditions** | Create resources only in some cases (e.g. only in prod) |
| **Outputs** | Values to show or **export** for other stacks to import |

**Features worth knowing**

- **Change sets**: preview exactly what an update will add, modify or replace before you run it (like `terraform plan`).
- **Rollback**: if creation or an update fails, the stack rolls back to the last good state automatically.
- **Drift detection**: finds resources that were changed manually outside CloudFormation.
- **StackSets**: deploy one template to **many accounts and Regions** in one operation (works with Organizations).
- **Nested stacks** reuse common components; **cross-stack references** share outputs via `Export` / `Fn::ImportValue`.
- **DeletionPolicy**: `Retain` or `Snapshot` keeps data (e.g. an RDS database) when the stack is deleted.
- **Intrinsic functions**: `Ref`, `Fn::GetAtt`, `Fn::Sub`, `Fn::Join`; **pseudo parameters** like `AWS::Region` and `AWS::AccountId`.
- **cfn-init / helper scripts** configure EC2 instances at launch; **creation policies / wait conditions** wait until they signal success.

!!! tip "Exam tip"
    "Replicate the same environment across Regions/accounts quickly and consistently" → **CloudFormation** (with **StackSets** for many accounts). "Preview changes" → **change sets**.

### AWS CDK (Cloud Development Kit)

- Define infrastructure in a **programming language you already know**: TypeScript, JavaScript, Python, Java, C#, Go.
- `cdk synth` generates a **CloudFormation template** under the hood; `cdk deploy` deploys it as a stack. (`cdk bootstrap` prepares an account/Region the first time.)
- Building blocks are **constructs**: L1 (raw CloudFormation resources), L2 (sensible defaults), L3 / patterns (whole architectures, e.g. ALB + Fargate service).
- Benefits: loops, conditions, type checking and IDE support, and reusable components shared as libraries.

---

## Monitoring & observability

### Amazon CloudWatch

Monitors your AWS resources and applications **in real time** by collecting **metrics** and **logs**, and acting on them with **alarms**.

| Feature | What it does |
|---|---|
| **Metrics** | Time series data from AWS services (EC2 CPU, ELB request count…) and **custom metrics** from your apps |
| **Alarms** | Watch a metric against a **threshold** and act: notify via **SNS**, trigger **EC2 Auto Scaling**, or stop/reboot/recover an EC2 instance |
| **Logs** | Collect, store and search logs in **log groups**; set retention; **Logs Insights** queries them |
| **Events** | Now **Amazon EventBridge**: react to state changes and schedules |
| **Dashboards** | Custom, shareable views of metrics, including cross-account and cross-Region |
| **Insights** | Container Insights (ECS/EKS), Lambda Insights, Application Signals, Contributor Insights |

**Facts that show up in questions**

- EC2 basic monitoring sends metrics every **5 minutes**; **detailed monitoring** every **1 minute** (paid).
- **Memory and disk usage are not collected by default**. Install the **CloudWatch agent** on the instance for these (and for OS/app logs).
- Alarm states: **OK**, **ALARM**, **INSUFFICIENT_DATA**. **Composite alarms** combine several alarms to reduce noise.
- Alarms drive **Auto Scaling policies** for EC2 (scale out when CPU > 70%, etc.).
- **Metric filters** turn log patterns (e.g. `ERROR`) into metrics you can alarm on.
- **Subscription filters** stream logs in real time to Lambda, Kinesis Data Streams, Firehose or OpenSearch.
- Export logs to **S3** for cheap long-term storage.
- **Synthetics canaries** test endpoints and user journeys on a schedule.

### AWS X-Ray

- **Distributed tracing**: follows a request as it travels through API Gateway, Lambda, ECS, EC2, DynamoDB and other services.
- Data model: a **trace** (the end-to-end request) is made of **segments** (one per service), which are made of **subsegments** (calls to downstream services, DB queries).
- **Service map** shows how services connect, with latency and error rates, so you can spot **bottlenecks and failing components**.
- **Sampling rules** control how many requests are traced; **annotations** are indexed key-value pairs you can filter traces by.
- Instrument with the X-Ray SDK or **AWS Distro for OpenTelemetry (ADOT)**; run the X-Ray daemon on EC2/ECS. Lambda and API Gateway have it built in (turn on active tracing).

!!! tip "Exam tip"
    "Which microservice is slowing requests down?" → **X-Ray**. "Alert when CPU is high" → **CloudWatch alarm**. "Who made the API call?" → **CloudTrail**.

### AWS Health Dashboard

- **Service health** (public events): the status of AWS services in every Region.
- **Your account health** (personal events): issues, **scheduled maintenance** (e.g. EC2 instance retirement) and notifications that affect **your** resources.
- Send events to **EventBridge** to automate a response (notify a team, start a runbook). **Organizational view** shows events for all accounts.

### Amazon Managed Service for Prometheus & Amazon Managed Grafana

- **Managed Prometheus (AMP)**: serverless, Prometheus-compatible **time series database** for container and application metrics. Query with **PromQL**; common with **EKS**. Same open-source tooling you'd use outside AWS, without running the servers.
- **Managed Grafana (AMG)**: managed Grafana workspaces to build **dashboards** from Prometheus, CloudWatch, X-Ray, OpenSearch and other sources. Sign-in via IAM Identity Center or SAML.

---

## Optimisation & best practices

### AWS Trusted Advisor

Checks your account against AWS best practices and gives **recommendations**.

- Categories: **cost optimisation**, **performance**, **security**, **fault tolerance**, **service limits (quotas)**, and **operational excellence**.
- Examples: idle load balancers, underused EC2 instances, open security group ports, S3 buckets with public access, no MFA on root, services close to their quota.
- **Basic / Developer support** get only core security checks and service quotas. **Business and Enterprise support** unlock **all checks**, the API, and organisation-wide views.

### AWS Compute Optimizer

- Uses **ML** on CloudWatch metrics (14 days by default) to find resources that are **over- or under-provisioned**, and recommends better sizes.
- Covers **EC2 instances, Auto Scaling groups, EBS volumes, Lambda functions, ECS services on Fargate, RDS** and commercial software licences.
- Shows the expected savings and performance risk of each recommendation.

!!! note "Trusted Advisor vs Compute Optimizer"
    Trusted Advisor = broad best-practice checks across many areas. Compute Optimizer = deep **right-sizing** recommendations for compute.

### AWS Launch Wizard

- Guided deployment of **third-party enterprise applications** using AWS best practices: **Microsoft SQL Server**, **SAP**, **Active Directory**, Exchange and more.
- You describe your needs; it sizes the resources, estimates cost and deploys them automatically (creating CloudFormation templates you can reuse).

---

## Multi-account governance

### AWS Organizations

Manage **many AWS accounts** centrally.

- One **management account** plus **member accounts**, grouped into **organisational units (OUs)** in a hierarchy (e.g. `Security`, `Prod`, `Dev`).
- **Consolidated billing**: one bill for all accounts, combined usage for **volume discounts**, and shared **Reserved Instance / Savings Plans** discounts.
- **Service control policies (SCPs)**: set the **maximum permissions** for accounts or OUs (e.g. deny leaving the Organization, allow only certain Regions).
    - SCPs **don't grant** permissions; IAM must still allow the action.
    - SCPs **don't apply to the management account**.
- **Resource control policies (RCPs)** set maximum permissions on resources (S3, KMS…); also **tag, backup and AI opt-out policies**.
- Turn on **organisation-wide services**: CloudTrail organisation trail, Config aggregator, GuardDuty and Security Hub with a delegated admin.

### AWS Control Tower

- The easy way to **set up and govern a secure multi-account environment** (a **landing zone**) following AWS best practices. Built **on top of AWS Organizations**, so it takes Organizations a step further.
- **Account Factory** creates new, pre-configured accounts in a standard way.
- **Controls (guardrails)**:

| Type | How it's enforced | Example |
|---|---|---|
| **Preventive** | **SCPs**: stop the action from happening | Disallow deleting CloudTrail logs |
| **Detective** | **AWS Config rules**: find and report violations | Detect unencrypted EBS volumes |
| **Proactive** | **CloudFormation hooks**: block non-compliant resources before they're created | Require S3 buckets to have encryption |

- A dashboard shows compliance across all accounts.

!!! tip "Exam tip"
    "Set up a new multi-account environment quickly with best-practice guardrails" → **Control Tower**. "Restrict what member accounts can do" → **SCPs in Organizations**.

### AWS Resilience Hub

- Helps you **define, assess and improve the resilience** of an application, mostly for **disaster recovery** planning.
- Flow: add your application (from CloudFormation, Terraform, EKS, etc.) → set a **resiliency policy** with **RTO and RPO** targets → run an **assessment** → get **recommendations** (Multi-AZ, backups, alarms, SOPs).
- Integrates with **AWS Fault Injection Service** to test that recovery actually works.

### AWS Resource Explorer

- **Search for resources** by name, tag or type across Regions (and across accounts in an Organization) from one search bar.
- Handy for inventory and finding forgotten resources.

### AWS Resource Access Manager (RAM)

- **Share AWS resources** with other accounts, OUs or the whole Organization, instead of creating copies in every account.
- Commonly shared: **VPC subnets** (VPC sharing), **Transit Gateways**, **Route 53 Resolver rules**, License Manager configurations, Aurora DB clusters, prefix lists.

!!! tip "Exam tip"
    "Several accounts must deploy into the same centrally managed VPC" → **VPC sharing with RAM**.

---

## Quick decision cheat sheet

| Scenario | Answer |
|---|---|
| Repeatable infrastructure from templates | CloudFormation |
| Preview infrastructure changes | CloudFormation change sets |
| Deploy one template to many accounts/Regions | CloudFormation StackSets |
| Keep an RDS DB when its stack is deleted | DeletionPolicy: Retain / Snapshot |
| Write infrastructure in Python/TypeScript | CDK |
| Alert or scale on a metric threshold | CloudWatch alarm |
| Monitor EC2 memory / disk usage | CloudWatch agent |
| Search application logs | CloudWatch Logs Insights |
| Stream logs to OpenSearch/Lambda in real time | CloudWatch Logs subscription filter |
| Find the slow service in a request path | X-Ray |
| Notified of AWS maintenance affecting your resources | Health Dashboard + EventBridge |
| Managed Prometheus metrics for EKS | Managed Service for Prometheus |
| Managed dashboards from many data sources | Managed Grafana |
| Best-practice checks (cost, security, quotas) | Trusted Advisor |
| Right-size EC2 / Lambda / EBS | Compute Optimizer |
| Deploy SQL Server / SAP with best practices | Launch Wizard |
| Central billing and policies for many accounts | Organizations |
| Restrict Regions or services for member accounts | SCPs |
| Multi-account landing zone with guardrails | Control Tower |
| Check an app meets its RTO/RPO | Resilience Hub |
| Find resources across Regions | Resource Explorer |
| Share subnets / Transit Gateway across accounts | Resource Access Manager |
