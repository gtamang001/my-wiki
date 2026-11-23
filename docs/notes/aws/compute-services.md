---
description: AWS exam notes on compute, covering EC2, Image Builder, ENIs, Elastic Beanstalk, Lightsail, ECS, EKS, ECR, App Runner, Batch, Lambda, SAM, Amplify and hybrid options such as Outposts and the Snow Family.
tags:
  - AWS
  - Compute
  - Exam Prep
---

# AWS Compute Services

Quick revision notes for the Solutions Architect exam. Compute services range from **full control** (EC2 virtual machines) through **containers** (ECS, EKS) to **serverless** (Lambda, Fargate), where AWS runs the servers for you. The table below gives one line per service; the sections after it go deeper on the ones that show up most in questions.

## At a glance

| Service | Category | One-liner |
|---|---|---|
| **EC2** | Virtual machines | Resizable virtual servers; you pick the instance type, OS and pricing model |
| **EC2 Image Builder** | VM images | Automates building, patching, testing and sharing AMIs and container images |
| **Elastic Network Interface (ENI)** | Networking | A virtual network card attached to an instance: private IP, MAC, security groups |
| **Elastic Beanstalk** | PaaS | Upload your code and it provisions EC2, Auto Scaling and a load balancer for you |
| **Lightsail** | Simple VPS | Low-cost virtual servers, databases and load balancers at a fixed monthly price |
| **ECS** | Containers | AWS's own container orchestrator; runs tasks on EC2 or Fargate |
| **EKS** | Containers | Managed **Kubernetes** control plane |
| **ECR** | Containers | Private (and public) Docker/OCI image registry |
| **App Runner** | Containers / PaaS | Source code or container image → running HTTPS web app, no infrastructure to manage |
| **Batch** | Batch jobs | Queues and runs batch jobs on managed EC2, Spot or Fargate capacity |
| **Lambda** | Serverless | Runs functions in response to events; pay per request and per millisecond |
| **SAM** | Serverless IaC | CloudFormation shorthand plus CLI to build, test and deploy serverless apps |
| **Serverless Application Repository** | Serverless | Catalogue of ready-made serverless apps you can deploy or publish |
| **Amplify** | Web & mobile | Build and host full-stack web/mobile apps with CI/CD from Git |
| **Outposts** | Hybrid | AWS-managed racks/servers in **your** data centre, same AWS APIs |
| **ECS / EKS Anywhere** | Hybrid | Run ECS or EKS on **your own** hardware, managed from AWS |
| **VMware Cloud on AWS** | Hybrid | Run your vSphere workloads on dedicated bare-metal hosts in AWS |
| **Snow Family (compute)** | Edge | Rugged Snowball Edge devices that run EC2 and Lambda where there's no connectivity |

---

## Virtual machines

### Amazon EC2 (Elastic Compute Cloud)

- **AMI** (Amazon Machine Image): the template for an instance (OS + software). AMIs are **Regional**; copy them to use in another Region.
- **User data**: a script that runs once at first boot (install packages, pull config).
- **Instance families**: General purpose (**M, T**), Compute optimised (**C**), Memory optimised (**R, X**), Storage optimised (**I, D**), Accelerated (**P, G, Inf, Trn**). T instances are burstable (CPU credits).
- **Hibernate**: saves RAM to the encrypted EBS root volume so the instance resumes fast.

**Purchasing options**

| Option | Best for | Discount |
|---|---|---|
| **On-Demand** | Short, unpredictable workloads | None |
| **Savings Plans** | Commit to $/hour for 1 or 3 years; Compute Savings Plans also cover Fargate and Lambda | Up to ~72% |
| **Reserved Instances** | Steady workloads on a specific instance type/Region for 1 or 3 years (Convertible lets you change type) | Up to ~72% |
| **Spot** | Fault-tolerant, flexible jobs (batch, CI, big data); **2-minute** interruption notice | Up to ~90% |
| **Dedicated Hosts** | Whole physical server; **bring-your-own licence** tied to sockets/cores, compliance | — |
| **Dedicated Instances** | Hardware not shared with other customers (no host-level control) | — |
| **Capacity Reservations** | Guarantee capacity in an AZ, no term commitment (billed whether used or not) | None |

**Placement groups**

| Type | What it does | Use for |
|---|---|---|
| **Cluster** | Packs instances close together in **one AZ** | Lowest latency, highest throughput (HPC) |
| **Spread** | Each instance on distinct hardware; max **7 per AZ** | A few critical instances that must not fail together |
| **Partition** | Groups of instances on separate racks (up to 7 partitions per AZ) | Large distributed systems: Hadoop, Cassandra, Kafka |

!!! tip "Exam tip"
    "Cheapest for a steady 24/7 workload" → **Savings Plans / Reserved Instances**. "Cheapest, can tolerate interruption" → **Spot**. "Licensing per socket/core" → **Dedicated Hosts**.

### EC2 Image Builder

- Pipeline that **builds, hardens, tests and distributes** AMIs (and container images) on a schedule or when base images change.
- Can share/copy the output to other accounts and Regions. You pay only for the underlying resources.

### Elastic Network Interface (ENI)

- A virtual NIC with a primary private IPv4, optional secondary IPs, one Elastic IP per private IP, a MAC address and security groups.
- **Bound to an AZ**. You can detach it and attach it to another instance for **fast failover** (the IP and MAC move with it).
- Related: **ENA** (Enhanced Networking, up to 100+ Gbps) and **EFA** (Elastic Fabric Adapter, low-latency HPC/MPI on Linux).

### AWS Elastic Beanstalk

- **Platform as a Service**: you upload code (Java, .NET, Node.js, Python, Go, Docker…); Beanstalk creates the EC2, Auto Scaling group, ELB and monitoring. You still have access to the resources.
- **Web server tier** (behind a load balancer) or **worker tier** (pulls from an SQS queue).
- Deployment modes: all-at-once, rolling, rolling with additional batch, **immutable**, traffic splitting. **Blue/green** = new environment + swap URLs.

### Amazon Lightsail

- Simple VPS bundles (VM, storage, data transfer) at a **predictable monthly price**, plus managed databases, containers and load balancers.
- For simple websites, dev/test and people new to AWS. Limited integration with the rest of AWS.

!!! tip "Exam tip"
    "Developer with little AWS experience needs a simple, low-cost website" → **Lightsail**. "Deploy a web app quickly without managing infrastructure, but keep access to it" → **Elastic Beanstalk**.

---

## Containers

### Amazon ECS (Elastic Container Service)

- AWS's native orchestrator. A **task definition** describes containers; a **service** keeps N tasks running and registers them with an ALB/NLB.
- **Launch types**: **EC2** (you manage the instances) or **Fargate** (serverless, you just set CPU/memory per task).
- **IAM**: the **task role** gives the containers AWS permissions; the task execution role lets ECS pull images and write logs.
- Scales with **Service Auto Scaling**; EC2 capacity scales with capacity providers.

### Amazon EKS (Elastic Kubernetes Service)

- Managed Kubernetes control plane across multiple AZs. Workers: **managed node groups**, self-managed nodes, or **Fargate**; Karpenter or Cluster Autoscaler for node scaling.
- Choose EKS when you need Kubernetes, its ecosystem, or portability across clouds/on-prem.

### Amazon ECR (Elastic Container Registry)

- Stores container images, integrated with IAM, ECS, EKS and Lambda.
- **Image scanning** (basic, or enhanced via Inspector), **lifecycle policies** to clean up old images, cross-Region/cross-account **replication**.

### AWS App Runner

- Point it at a container image or source repo; it builds, deploys, load-balances, scales and gives you an HTTPS URL.
- Simplest way to run a containerised **web app or API**; no clusters, no load balancers to configure.

### AWS Batch

- Submit jobs to **job queues**; Batch provisions **compute environments** (EC2, Spot or Fargate) and runs them.
- No 15-minute limit like Lambda, so it suits long-running jobs, simulations and big data processing.

!!! tip "Exam tip"
    "Containers without managing servers" → **Fargate** (with ECS or EKS). "Already use Kubernetes" → **EKS**. "Long-running batch jobs on Spot" → **Batch**.

---

## Serverless

### AWS Lambda

- Runs code in response to events (S3, SQS, DynamoDB Streams, API Gateway, EventBridge…). Pay per request and duration.

| Limit | Value |
|---|---|
| Max run time | **15 minutes** |
| Memory | 128 MB – 10,240 MB (CPU scales with memory) |
| `/tmp` storage | Up to 10 GB |
| Concurrency | 1,000 per Region by default (can be raised) |
| Package size | 50 MB zipped / 250 MB unzipped, or a **container image up to 10 GB** |

- **Reserved concurrency**: caps (and guarantees) a function's concurrency. **Provisioned concurrency**: pre-warmed instances to avoid **cold starts**. **SnapStart** also cuts cold starts (Java, Python, .NET).
- **In a VPC**: Lambda gets ENIs in your subnets; it needs a **NAT gateway** or VPC endpoints to reach the internet/AWS services.
- Async invocations: retries, then **DLQ** or **destinations** for failures.
- **Function URLs** give a function its own HTTPS endpoint without API Gateway.

!!! tip "Exam tip"
    "Job runs longer than 15 minutes" → not Lambda; use **Fargate, Batch or Step Functions**. "Cold starts are a problem" → **provisioned concurrency**.

### AWS SAM (Serverless Application Model)

- Extension of CloudFormation (`Transform: AWS::Serverless-2016-10-31`) with short resource types for functions, APIs and tables.
- **SAM CLI**: `sam build`, `sam local invoke` (test locally), `sam deploy`. Supports gradual Lambda deployments with **CodeDeploy** (canary, linear).

### AWS Serverless Application Repository (SAR)

- A catalogue of published serverless apps (built with SAM). Deploy them into your account, or publish your own to share privately, cross-account or publicly.

### AWS Amplify

- Tools and hosting for **full-stack web and mobile apps**: Amplify Hosting (static/SSR sites with Git-based CI/CD) plus backend features built on Cognito, AppSync, DynamoDB and S3.

---

## Hybrid & edge

### AWS Outposts

- AWS-owned **racks or servers installed in your data centre**, managed by AWS; you run EC2, EBS, ECS, EKS, RDS and S3 on them with the same APIs.
- For **low latency** to on-prem systems, **local data processing** and **data residency**. You're responsible for physical security of the site.

### ECS Anywhere / EKS Anywhere

- **ECS Anywhere**: register your own servers/VMs with an ECS cluster and manage them from the ECS console.
- **EKS Anywhere**: run EKS-compatible Kubernetes clusters on your own infrastructure (vSphere, bare metal).
- Unlike Outposts, **you supply and manage the hardware**.

### VMware Cloud on AWS

- Runs the VMware stack (vSphere, vSAN, NSX) on dedicated bare-metal hosts in AWS. Move VMs without re-platforming, extend a data centre, or use it for DR.
- Now sold by **Broadcom** rather than AWS, but still appears in exam questions about migrating VMware workloads.

### Snow Family (compute)

- **Snowball Edge Compute Optimized** runs EC2 instances and Lambda functions at disconnected or remote sites (ships, mines, field sites), then ships data back to AWS.
- See [Migration & transfer](migration-services.md) for the data-transfer side and current availability.

---

## Quick decision cheat sheet

| Scenario | Answer |
|---|---|
| Full control of OS, steady 24/7 workload, lowest cost | EC2 + Savings Plans / Reserved Instances |
| Interruptible batch or CI jobs, lowest cost | EC2 Spot (or Batch on Spot) |
| BYOL licensed per socket/core | Dedicated Hosts |
| Lowest latency between instances (HPC) | Cluster placement group |
| Keep a few critical instances on separate hardware | Spread placement group |
| Automate patched, hardened AMIs | EC2 Image Builder |
| Fail over an IP/MAC to a standby instance | Move the ENI |
| Deploy a web app without managing infrastructure | Elastic Beanstalk |
| Simple, fixed-price website for a beginner | Lightsail |
| Run containers without managing servers | ECS or EKS on Fargate |
| Need Kubernetes | EKS |
| Container web app/API with the least setup | App Runner |
| Event-driven code under 15 minutes | Lambda |
| Long-running job or > 15 minutes | Fargate, Batch or Step Functions |
| Define and test serverless apps locally | SAM |
| Front-end hosting with Git CI/CD | Amplify Hosting |
| AWS services inside your own data centre | Outposts |
| Containers on your own hardware, managed from AWS | ECS Anywhere / EKS Anywhere |
| Migrate VMware VMs without changing them | VMware Cloud on AWS |
| Compute at a site with no connectivity | Snowball Edge Compute Optimized |
