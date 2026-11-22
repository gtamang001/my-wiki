---
description: AWS exam notes on migration and transfer, covering Migration Hub, Application Discovery Service, Application Migration Service, DMS, Elastic Disaster Recovery, Mainframe Modernization, DataSync, Transfer Family and the Snow Family.
tags:
  - AWS
  - Migration
  - Exam Prep
---

# AWS Migration & Transfer Services

Quick revision notes for the Solutions Architect exam. These services help you **plan** a migration, **move** servers, databases and files into AWS, and **recover** from disasters. The table below gives one line per service; the sections after it go deeper on the ones that show up most in questions.

## At a glance

| Service | Moves / does | One-liner |
|---|---|---|
| **Migration Hub** | Planning & tracking | One place to discover, plan and track migrations across AWS tools |
| **Application Discovery Service** | Discovery | Collects server inventory, usage and dependencies from on-prem (agent or agentless) |
| **Application Migration Service (MGN)** | Servers | Lift-and-shift rehosting of servers into EC2 with continuous replication |
| **Database Migration Service (DMS)** | Databases | Migrates databases to AWS with minimal downtime; Schema Conversion for engine changes |
| **Elastic Disaster Recovery (DRS)** | Servers (DR) | Continuously replicates servers to AWS and recovers them within minutes |
| **Mainframe Modernization** | Mainframe apps | Migrate and modernise mainframe (COBOL) applications to AWS |
| **DataSync** | Files / objects | Automated online transfer of files between on-prem storage and S3, EFS or FSx (and between AWS services) |
| **Transfer Family** | Files over FTP | Managed SFTP / FTPS / FTP / AS2 servers that store files in S3 or EFS |
| **Snow Family** | Files offline | Rugged physical devices for moving large data offline and for edge computing |

### Picking a migration service

| What you're moving | Service |
|---|---|
| Physical / virtual servers, as-is (rehost) | **Application Migration Service** |
| Databases (same or different engine) | **DMS** (+ Schema Conversion for different engines) |
| Files/objects over the network, scheduled | **DataSync** |
| Partners sending files via SFTP/FTP | **Transfer Family** |
| Petabytes with slow or no network | **Snowball Edge** |
| Mainframe applications | **Mainframe Modernization** |
| DR for on-prem or cloud servers | **Elastic Disaster Recovery** |

---

## Plan & discover

### AWS Migration Hub

- **Centralised** place to **plan and track** migrations across AWS and partner tools.
- Uses discovery data (agents or agentless) to build an **inventory** of servers and their **compute usage**, group them into applications, and generate **EC2 right-sizing and estimated cost** reports.
- **Strategy Recommendations** suggest a migration strategy (rehost, replatform, refactor…) per application.

!!! note "Availability"
    AWS is moving migration planning and tracking to **AWS Transform**, which uses agentic AI to assess and migrate workloads. Migration Hub still appears in exam material.

### AWS Application Discovery Service

- Collects data about your **on-premises servers**: configuration, utilisation (CPU, memory, disk), running processes and **network connections**, so it can map **dependencies** between servers.
- Two ways to collect:
    - **Agentless** (Agentless Collector for VMware vCenter): VM inventory and performance data.
    - **Agent-based** (Discovery Agent on each server): more detail, including processes and network dependencies.
- Data feeds Migration Hub for grouping and recommendations; export to S3/Athena for analysis.

---

## Move servers & databases

### AWS Application Migration Service (MGN)

The main **lift-and-shift (rehost)** service. It simplifies and speeds up migration and **cuts the time and errors** compared with doing it by hand.

- Install the replication agent on the source server (physical, VMware, Hyper-V, or another cloud). It does **continuous block-level replication** into a low-cost **staging area** in your AWS account.
- Launch **test instances** at any time to check the app works **before cutover**, without stopping the source.
- At **cutover**, MGN launches the production EC2 instances from the latest replicated data, so downtime is **minutes**.
- Replaces the older **Server Migration Service (SMS)** and CloudEndure Migration.

!!! tip "Exam tip"
    "Lift and shift servers to EC2 with minimal downtime" → **Application Migration Service**.

### AWS Database Migration Service (DMS)

Think of it as the **database version of Application Migration Service**: it moves databases from on-prem (or other clouds) into AWS.

- Supports **20+ engines** as sources and targets: Oracle, SQL Server, MySQL, PostgreSQL, MongoDB, SAP and more. Targets include RDS, Aurora, Redshift, DynamoDB, S3, OpenSearch, Kinesis.
- The **source database stays online** during migration, keeping downtime to a minimum.
- Migration types: **full load**, **full load + CDC** (change data capture keeps the target in sync until cutover), or **CDC only** for continuous replication.
- Runs on a **replication instance** (or **DMS Serverless**).
- **Homogeneous** (same engine, e.g. MySQL → RDS MySQL): just DMS.
- **Heterogeneous** (different engines, e.g. Oracle → Aurora PostgreSQL): first convert the schema and code with **DMS Schema Conversion** or the **Schema Conversion Tool (SCT)**, then move the data with DMS.

!!! tip "Exam tip"
    "Migrate Oracle to Aurora" or "change engine" → **SCT / DMS Schema Conversion + DMS**. "Keep the target in sync until cutover" → **DMS with CDC**.

### AWS Elastic Disaster Recovery (DRS)

- **Continuous, block-level replication** of servers (on-prem, other clouds, or other AWS Regions) into a low-cost **staging subnet** in AWS.
- Recover to **any point in time**, with **RPO of seconds** and **RTO of minutes**.
- Run **non-disruptive DR drills** whenever you want, without affecting the source.
- After the incident is resolved, **fail back** to on-prem (or the original Region).
- Launches recovered servers on **EC2**, using EBS snapshots; works with ELB and other AWS services in the recovery design.

!!! note "MGN vs DRS"
    Both use the same replication technology. **MGN** is for a one-time **migration** (cutover, then done). **DRS** keeps replicating for ongoing **disaster recovery**.

### AWS Mainframe Modernization

- Migrate and modernise **mainframe applications** (COBOL, PL/I, CICS, JCL) to AWS managed runtimes.
- Two main patterns:
    - **Replatform**: run the existing code on AWS with minimal changes (Rocket Software / Micro Focus runtime).
    - **Refactor**: **automatically convert** COBOL to **Java** (AWS Blu Age) to get a modern, cloud-native app.
- Includes tools for analysis, testing and data migration. AWS Transform for mainframe adds AI-assisted code analysis and conversion.

---

## Move files & data

### AWS DataSync

Automated, **online** data transfer service that is up to 10x faster than open-source tools.

- **Components**:
    - **Locations**: the **source** and **destination**: NFS, SMB, HDFS, self-managed object storage, other clouds, **S3** (any storage class), **EFS**, **FSx**.
    - **Agent**: a VM you deploy on-prem to read local storage. **Not needed** for AWS-to-AWS transfers.
    - **Tasks**: define what to copy, filters, schedule and options.
- **Task execution phases**: Queued → Launching → Preparing → Transferring → Verifying.
- Preserves **metadata and permissions**, **verifies** data integrity, encrypts in transit, can **throttle bandwidth**, and runs on a **schedule**.
- **Discovery** analyses on-prem storage systems and recommends AWS storage targets.
- Also syncs data **between AWS services and Regions** (e.g. EFS in one Region to EFS in another).

!!! tip "Exam tip"
    "Move/sync large file shares (NFS/SMB) to S3 or EFS over the network, scheduled" → **DataSync**. If there's no usable network (or it would take weeks) → **Snowball Edge**.

### AWS Transfer Family

- Fully managed **SFTP, FTPS, FTP and AS2** servers whose storage is **S3 or EFS**.
- Partners and customers keep using their existing file transfer clients and scripts. No servers to run.
- Users authenticate with service-managed keys, **Active Directory**, or a custom identity provider (Lambda/API Gateway).
- Plain **FTP** is only allowed inside a VPC (not over the internet).

### AWS Snow Family

**Physical devices** that AWS ships to you for **offline data transfer** when the network is too slow, too expensive or unavailable. Devices are **rugged, tamper-resistant and encrypted** (KMS, 256-bit keys); you copy data on, ship it back, and AWS loads it into S3.

| Device | Size | Use |
|---|---|---|
| **Snowball Edge Storage Optimized** | Suitcase-sized, ~50 lb | Large migrations (tens to hundreds of TB per device); local storage |
| **Snowball Edge Compute Optimized** | Suitcase-sized | More vCPUs/GPU for **edge computing** in disconnected or harsh places (ships, mines, military sites) |
| **Snowcone** | Small, ~4.5 lb | Small data and edge computing (retired) |
| **Snowmobile** | 45-foot shipping container | Exabyte-scale moves (retired) |

- Both Snowball Edge types can **run EC2 instances and Lambda functions** at the edge to process data before it goes to AWS.
- Managed with **AWS OpsHub** (a desktop app) or the Snowball client.

!!! warning "Availability"
    AWS has retired **Snowmobile** and **Snowcone**, and is pointing new customers to online options (DataSync, Direct Connect, Data Transfer Terminal). The Snow Family still comes up in exam questions: "petabytes, limited bandwidth" → **Snowball Edge**.

!!! tip "Rule of thumb"
    If moving the data over your network would take **more than about a week**, consider a Snow device.

---

## Quick decision cheat sheet

| Scenario | Answer |
|---|---|
| Track a migration across many tools | Migration Hub |
| Map on-prem server dependencies | Application Discovery Service (agent-based) |
| VMware VM inventory without installing agents | Application Discovery Service agentless collector |
| Lift and shift servers to EC2, minimal downtime | Application Migration Service (MGN) |
| Migrate a database with minimal downtime | DMS (full load + CDC) |
| Change database engine (Oracle → Aurora PostgreSQL) | DMS Schema Conversion / SCT + DMS |
| Ongoing DR for on-prem servers, RPO seconds | Elastic Disaster Recovery |
| Test DR without affecting production | Elastic Disaster Recovery drills |
| Move COBOL mainframe apps to AWS | Mainframe Modernization |
| Scheduled sync of NFS/SMB shares to S3/EFS/FSx | DataSync |
| Copy data between EFS file systems across Regions | DataSync |
| Partners upload files via SFTP into S3 | Transfer Family |
| Petabytes of data, limited bandwidth | Snowball Edge Storage Optimized |
| Compute in a remote site with no connectivity | Snowball Edge Compute Optimized |
