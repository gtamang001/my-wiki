---
description: AWS exam notes on security, identity and compliance, covering IAM, Identity Center, Cognito, CloudTrail, Config, GuardDuty, Inspector, Macie, KMS, WAF, Shield and Network Firewall.
tags:
  - AWS
  - Security
  - Exam Prep
---

# AWS Security Services

Quick revision notes for the Solutions Architect exam. Security services help you **centralise control**, **enforce least privilege**, **detect threats** and **pass audits**. The table below gives one line per service; the sections after it go deeper on the ones that show up most in questions.

## At a glance

| Service | Category | One-liner |
|---|---|---|
| **IAM** | Identity | Users, groups, roles and policies that control who can do what in an AWS account |
| **IAM Identity Center** | Identity | Single sign-on for your **workforce** across many AWS accounts and business apps |
| **Cognito** | Identity | Sign-up/sign-in for **your app's customers**, plus temporary AWS credentials for them |
| **Directory Service** | Identity | Managed Microsoft Active Directory, or a proxy to your on-prem AD |
| **Verified Permissions** | Identity | Managed **authorization** (fine-grained permissions) for your own applications, using Cedar policies |
| **CloudTrail** | Audit | Records **who did what, when** (every API call from Console, CLI and SDK) |
| **Config** | Audit / compliance | Tracks **resource configuration** over time and flags non-compliant resources |
| **Artifact** | Compliance | Self-service portal for AWS compliance reports (SOC, PCI, ISO) and agreements |
| **GuardDuty** | Threat detection | ML-based threat detection from CloudTrail, VPC Flow Logs, DNS logs and more |
| **Inspector** | Vulnerability scanning | Automatically scans EC2, container images and Lambda for CVEs and network exposure |
| **Macie** | Data protection | Finds **sensitive data (PII)** in S3 buckets using ML and pattern matching |
| **Security Hub** | Posture management | One dashboard for security findings plus automated best-practice checks |
| **Security Lake** | Security data | Centralises security logs from AWS and third parties into an S3 data lake |
| **KMS** | Encryption | Create and manage encryption keys, integrated with almost every AWS service |
| **CloudHSM** | Encryption | Dedicated, single-tenant hardware security modules where **only you** control the keys |
| **ACM** | Certificates | Free public TLS certificates with automatic renewal for ELB, CloudFront and API Gateway |
| **Private CA** | Certificates | Your own private certificate authority for internal TLS certificates |
| **Secrets Manager** | Secrets | Stores and **automatically rotates** secrets such as DB passwords and API keys |
| **Security groups & NACLs** | Network | Instance-level (stateful) and subnet-level (stateless) firewalls in a VPC |
| **WAF** | Network / app | Layer 7 web application firewall that blocks SQL injection, XSS and bad bots |
| **Shield** | Network | DDoS protection. Standard is free; Advanced adds 24/7 response and cost protection |
| **Network Firewall** | Network | Managed stateful firewall with deep packet inspection at the VPC level |
| **Firewall Manager** | Network / governance | Centrally manage WAF, Shield, security group and firewall rules across an Organization |

---

## Identity & access

### AWS IAM (Identity and Access Management)

Controls **authentication** (who you are) and **authorization** (what you're allowed to do) in an AWS account. IAM is global and free.

| Building block | What it is |
|---|---|
| **User** | A person or app with long-term credentials (password and/or access keys) |
| **Group** | A collection of users; attach policies once for everyone. A user can belong to **multiple groups**; groups can't contain groups |
| **Role** | An identity with **temporary credentials** (via STS) that users, services or other accounts **assume**. Preferred over access keys |
| **Policy** | A JSON document that allows or denies actions on resources |

- **Policy types**: identity-based (attached to users/groups/roles), **resource-based** (e.g. S3 bucket policy, can grant cross-account access), **permissions boundaries** (maximum permissions an identity can have), **SCPs** (account-level limits from AWS Organizations), session policies.
- **Evaluation logic**: everything is **denied by default**; an explicit **Allow** grants access; an explicit **Deny always wins**.
- **Best practices**: lock away the **root user** (MFA, no access keys), follow **least privilege**, use **roles for EC2/Lambda** instead of storing keys, enable **MFA**, use **IAM Access Analyzer** to find unused or externally shared access.
- **Cross-account access**: create a role in account B with a trust policy for account A, and let A assume it.

!!! tip "Exam tip"
    "Give an EC2 instance or Lambda function access to S3" → **IAM role**, never access keys on the instance. "Access in another account" → **assume a cross-account role**.

### IAM Identity Center (successor to AWS SSO)

- **Workforce** single sign-on: one login for employees across **all accounts in AWS Organizations** and business apps (Salesforce, Microsoft 365…).
- Identity source: its built-in directory, **Active Directory**, or an external IdP (Okta, Microsoft Entra ID) via SAML 2.0 / SCIM.
- **Permission sets** define what a user or group can do in each account.

### Amazon Cognito

Authentication for **your application's end users** (customers, mobile/web app users), not your staff.

| Component | What it does |
|---|---|
| **User pools** | User directory for sign-up / sign-in. Returns **JWT tokens**; supports MFA and social/SAML login (Google, Facebook, Apple). Integrates with ALB and API Gateway |
| **Identity pools** | Exchange a login (user pool, social, or guest) for **temporary AWS credentials**, so users can access AWS resources like S3 or DynamoDB directly |

!!! tip "Exam tip"
    "Millions of mobile/web app users need to sign in" → **Cognito**. "Employees need SSO across AWS accounts" → **IAM Identity Center**.

### AWS Directory Service

Managed Microsoft Active Directory options on AWS:

| Mode | How it works | Use |
|---|---|---|
| **AWS Managed Microsoft AD** | A real Microsoft AD run by AWS; can set up a **trust** with your on-prem AD | AD-aware apps in AWS, MFA, the most features |
| **AD Connector** | A **proxy** that redirects requests to your **on-prem AD**; no directory data cached in AWS | Use existing on-prem users to log in to AWS apps |
| **Simple AD** | Cheap, Samba-based AD-compatible directory | Small, basic needs; **can't** connect to an on-prem AD |

### Amazon Verified Permissions

- Managed **authorization service for your own applications**: decide "can this user do this action on this resource?"
- Policies are written in **Cedar** (open-source policy language) and kept outside your app code.
- Supports role-based and attribute-based access control; pairs with Cognito for authentication.

---

## Audit, logging & compliance

### AWS CloudTrail

The **audit trail** of your AWS account: records API calls made through the Console, CLI, SDK and AWS services.

- **Event history** shows the last **90 days** of management events for free.
- Create a **trail** to keep events longer: deliver to **S3** (long-term storage) and/or **CloudWatch Logs** (search and alarms).
- **Event types**: **management events** (create/delete/modify resources, on by default), **data events** (S3 object-level, Lambda invoke; high volume, off by default), **Insights events** (unusual API activity).
- **Organization trail**: one trail for every account in AWS Organizations.
- **Log file integrity validation** proves the logs weren't tampered with. Encrypt with KMS.
- **CloudTrail Lake**: store and query events with SQL.

!!! tip "Exam tip"
    "Who deleted the resource / who changed the security group?" → **CloudTrail**. "What did the resource look like before and is it compliant?" → **Config**.

### AWS Config

Tracks **configuration changes** to your AWS resources and checks them against rules.

- Keeps an **inventory** and a **configuration history** of resources over time.
- **Config rules**: AWS-managed (e.g. "S3 buckets must block public access") or **custom** (Lambda). Report **non-compliant** resources.
- **Remediation**: fix issues automatically with **Systems Manager Automation** documents.
- **Conformance packs** bundle rules into a compliance framework; **aggregators** give a multi-account, multi-Region view.
- Config **detects and reports**; it doesn't **prevent** changes. Use SCPs or IAM to prevent them.

### AWS Artifact

- Self-service portal to download **AWS compliance reports** (SOC 1/2/3, PCI DSS, ISO certifications) for your auditors.
- Review and accept **agreements** such as the HIPAA **BAA** for your account or Organization.

---

## Threat detection & posture

### Amazon GuardDuty

**Intelligent threat detection** that runs continuously with no agents or infrastructure for you to manage.

- Uses **ML, anomaly detection and threat intelligence** on: **CloudTrail** events, **VPC Flow Logs**, **DNS logs**, plus optional EKS audit logs, S3 data events, RDS login activity, Lambda network activity, runtime monitoring and **malware protection** for EBS/S3.
- Finding types include **reconnaissance** (port scans), **instance compromise** (crypto-mining, talking to known bad IPs), **account compromise** (unusual API calls from odd locations), and **bucket compromise** (suspicious S3 access).
- Send findings to **EventBridge** to automate a response (e.g. Lambda isolates the instance).
- Manage across an Organization with a **delegated administrator** account.

### Amazon Inspector

**Automated, continuous vulnerability management.**

- Scans **EC2 instances** (via the SSM agent, or agentless), **container images in ECR**, and **Lambda functions** for known software vulnerabilities (**CVEs**) and unintended **network exposure**.
- Rescans automatically when a new CVE is published or the workload changes.
- Gives a risk score per finding; sends findings to Security Hub and EventBridge.

!!! note "Inspector Classic"
    The older flow you might see in study material (set up agent → define targets → define assessment templates → run assessment → review and remediate) is **Inspector Classic**. The current Inspector runs automatically once enabled.

### Amazon Macie

- Uses **ML and pattern matching** to discover **sensitive data** such as **PII**, financial data and credentials in **S3**.
- Also reports bucket security risks (public, unencrypted, shared outside the account).
- Findings go to EventBridge and Security Hub for alerts and automation.

!!! tip "Exam tip"
    "Find PII in S3" → **Macie**. "Threats / suspicious activity" → **GuardDuty**. "Software vulnerabilities in EC2/containers" → **Inspector**.

### AWS Security Hub

- **Central security dashboard**: collects and prioritises findings from **GuardDuty, Inspector, Macie, Config, IAM Access Analyzer, Firewall Manager** and partner tools in one format (ASFF).
- Runs automated **compliance checks** against standards such as **AWS Foundational Security Best Practices, CIS, PCI DSS** and NIST. Requires **AWS Config** to be enabled.
- Aggregates across accounts and Regions; automate responses via EventBridge.

### Amazon Security Lake

- Automatically **centralises security logs** (CloudTrail, VPC Flow Logs, Route 53, Security Hub, third-party sources) into an **S3 data lake** in your account.
- Normalises data to the open **OCSF** schema, so analytics and SIEM tools can query it (e.g. with Athena or OpenSearch).

---

## Encryption, keys & secrets

### AWS KMS (Key Management Service)

Managed service to **create and control encryption keys**; integrated with S3, EBS, RDS, DynamoDB and most other services. Keys are **Regional** and never leave KMS unencrypted.

| Key type | Who manages it | Notes |
|---|---|---|
| **AWS owned keys** | AWS | Used by services by default; you can't see them |
| **AWS managed keys** (`aws/s3`, `aws/ebs`…) | AWS, in your account | Free; rotated automatically every year |
| **Customer managed keys** | You | Full control of the key policy, rotation and deletion; can share cross-account |

- **Symmetric** keys (AES-256, the default; one key encrypts and decrypts) and **asymmetric** keys (RSA/ECC key pairs for signing or encrypting outside AWS).
- Direct `Encrypt` is limited to **4 KB**. For **large data**, use **envelope encryption**: KMS generates a data key (`GenerateDataKey`), you encrypt the data with it, and store the encrypted data key alongside.
- Access is controlled by **key policies** (plus IAM and grants). Every key use is logged in CloudTrail.
- **Multi-Region keys** let you decrypt in another Region without re-encrypting.
- Deleting a key has a **7–30 day** waiting period.

### AWS CloudHSM

- **Dedicated, single-tenant hardware security modules** (FIPS 140-3 Level 3) in your VPC. **You** manage the keys and users; AWS can't access them.
- Deploy as a **cluster across AZs** for high availability.
- Use for strict compliance requirements, SSL/TLS offload, Oracle TDE, or as a **KMS custom key store**.

!!! tip "Exam tip"
    "Managed keys, integrated with AWS services" → **KMS**. "Single-tenant HSM, customer has exclusive control, FIPS 140 Level 3" → **CloudHSM**.

### AWS Certificate Manager (ACM)

- Issues **free public TLS certificates** and **renews them automatically**. You can also import third-party certificates (no auto-renewal).
- Deploys to **ELB, CloudFront, API Gateway** (and others). You **can't** export ACM public certificates onto your own EC2 web server.
- Certificates for **CloudFront** must be created in **us-east-1**.

### AWS Private Certificate Authority

- Managed **private CA** that issues certificates for **internal** services, devices and users (mTLS, internal websites) that the public shouldn't trust.
- Works with ACM to issue and renew private certificates.

### AWS Secrets Manager

- Stores secrets (DB passwords, API keys, tokens) encrypted with KMS.
- **Automatic rotation** using Lambda, with native support for **RDS, Aurora, Redshift and DocumentDB**.
- **Cross-Region replication** of secrets for DR.

!!! note "Secrets Manager vs Parameter Store"
    **SSM Parameter Store** also stores secrets (SecureString) and is cheaper or free, but has **no built-in rotation**. "Rotate DB credentials automatically" → **Secrets Manager**.

---

## Network & application protection

### Security groups & NACLs

| | **Security group** | **Network ACL (NACL)** |
|---|---|---|
| Applies to | Instance / ENI level | **Subnet** level |
| State | **Stateful**: return traffic allowed automatically | **Stateless**: return traffic needs its own rule |
| Rules | **Allow only** | **Allow and Deny** |
| Evaluation | All rules evaluated | Rules processed **in number order**, first match wins |
| Default | Denies all inbound, allows all outbound | Default NACL allows all |

!!! tip "Exam tip"
    "Block a specific IP address" → **NACL** (security groups can't deny). For apps behind CloudFront/ALB, **WAF** IP sets are usually the better answer.

### AWS WAF (Web Application Firewall)

- **Layer 7** firewall that inspects **HTTP(S) requests**.
- You build a **web ACL** with rules: **IP sets**, **rate-based rules** (throttle floods), **geo match**, and **managed rule groups** that block **SQL injection, cross-site scripting (XSS)**, bad bots and known exploits.
- Attach to **CloudFront, ALB, API Gateway, AppSync, Cognito user pools** and App Runner. **Not** to NLB (use Network Firewall or Shield there).

### AWS Shield

| Tier | Cost | Protection |
|---|---|---|
| **Shield Standard** | Free, on for everyone | Common Layer 3/4 DDoS attacks (SYN floods, UDP reflection) |
| **Shield Advanced** | Paid subscription (~$3,000/month per Organization) | More sophisticated attacks on EC2, ELB, CloudFront, Global Accelerator, Route 53; **24/7 Shield Response Team**; **cost protection** for scaling during an attack; WAF included for protected resources |

### AWS Network Firewall

- Managed, **stateful** network firewall and intrusion prevention for your **VPC**, with **deep packet inspection**.
- Deployed as **firewall endpoints** in a **dedicated firewall subnet**; route tables send traffic through them.
- Two rule engines: **stateless** (fast, simple packet filters) and **stateful** (**Suricata-compatible** IPS rules, **domain-name filtering**).
- Deployment models: **distributed** (a firewall in each VPC), **centralised** (an inspection VPC behind **Transit Gateway**), or combined.
- Covers traffic WAF doesn't: non-HTTP protocols, egress filtering, VPC-to-VPC traffic.

### AWS Firewall Manager

- **Central management** of security rules across all accounts in **AWS Organizations**: WAF rules, **Shield Advanced**, **security groups**, **Network Firewall** and Route 53 Resolver DNS Firewall.
- New accounts and resources get the policies **automatically**, so nothing is left unprotected.
- Requires AWS Organizations and AWS Config.

---

## Quick decision cheat sheet

| Scenario | Answer |
|---|---|
| Give EC2/Lambda access to AWS services | IAM role |
| Access resources in another account | Cross-account IAM role |
| Limit what accounts in an Organization can do | Service control policies (SCPs) |
| Cap the maximum permissions of a user/role | Permissions boundary |
| Employee SSO across many AWS accounts | IAM Identity Center |
| Sign-up / sign-in for app customers | Cognito user pools |
| App users need temporary AWS credentials | Cognito identity pools |
| Use on-prem AD users in AWS without syncing | AD Connector |
| Fine-grained permissions inside your own app | Verified Permissions |
| Who made this API call? | CloudTrail |
| Keep API logs longer than 90 days | CloudTrail trail to S3 |
| Track config changes / compliance rules | AWS Config |
| Download SOC/PCI reports for auditors | Artifact |
| Detect compromised instances or accounts | GuardDuty |
| Scan EC2/ECR/Lambda for CVEs | Inspector |
| Find PII in S3 | Macie |
| Single view of security findings and best-practice checks | Security Hub |
| Centralise security logs in a data lake | Security Lake |
| Encrypt data with managed keys | KMS |
| Encrypt data larger than 4 KB | Envelope encryption with KMS data keys |
| Single-tenant HSM, customer-only key control | CloudHSM |
| Free TLS certificate for ALB/CloudFront | ACM |
| Internal/private certificates | Private CA |
| Rotate DB passwords automatically | Secrets Manager |
| Deny a specific IP at the subnet level | NACL |
| Block SQL injection / XSS | WAF |
| DDoS protection with 24/7 response team | Shield Advanced |
| Deep packet inspection / egress domain filtering in a VPC | Network Firewall |
| Enforce firewall rules across all accounts | Firewall Manager |
