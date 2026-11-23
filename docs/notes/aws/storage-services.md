---
description: AWS exam notes on storage, covering EBS, instance store, EFS, FSx for Windows and Lustre, S3 storage classes, versioning and access points, AWS Backup, Elastic Disaster Recovery and Storage Gateway.
tags:
  - AWS
  - Storage
  - Exam Prep
---

# AWS Storage Services

Quick revision notes for the Solutions Architect exam. Storage questions usually come down to **block vs file vs object**, how much **durability and sharing** you need, and the **cheapest class** for how often you read the data. The table below gives one line per service; the sections after it go deeper on the ones that show up most in questions.

## At a glance

| Service | Type | One-liner |
|---|---|---|
| **EBS** | Block | Network-attached disk for one EC2 instance, bound to an AZ, persists after stop |
| **Instance Store** | Block | Physical disk on the host: very fast, but **data is lost** on stop/terminate |
| **EFS** | File (NFS) | Managed, elastic **Linux** file system shared by many instances across AZs |
| **FSx for Windows File Server** | File (SMB) | Managed **Windows** file shares with Active Directory integration |
| **FSx for Lustre** | File (HPC) | High-performance parallel file system for HPC/ML, can link to S3 |
| **S3** | Object | Unlimited, highly durable object storage with storage classes for every access pattern |
| **AWS Backup** | Backup | Central, policy-based backups across many AWS services and accounts |
| **Elastic Disaster Recovery** | DR | Continuous replication of servers to AWS for recovery in minutes |
| **Storage Gateway** | Hybrid | On-prem appliance that gives local apps access to cloud storage (file, volume, tape) |

---

## Block storage

### Amazon EBS (Elastic Block Store)

- A network drive for EC2. **Locked to one AZ**; to move it, take a **snapshot** and restore in another AZ/Region.
- Usually attached to one instance; **Multi-Attach** (io1/io2, same AZ) lets up to 16 Nitro instances share a volume.
- Root volume is deleted on termination by default; extra volumes are kept (controlled by *DeleteOnTermination*).

| Volume type | Kind | Use for |
|---|---|---|
| **gp3** | General purpose SSD | Default choice; 3,000 IOPS baseline, IOPS and throughput set independently |
| **gp2** | General purpose SSD | Older; IOPS grow with size (3 IOPS/GB) |
| **io2 Block Express / io1** | Provisioned IOPS SSD | Databases needing sustained high IOPS (up to 256,000) and low latency |
| **st1** | Throughput HDD | Big, sequential workloads: data warehouses, logs |
| **sc1** | Cold HDD | Lowest cost, infrequently accessed data |

- Only **SSD** types can be boot volumes.
- **Encryption** with KMS covers data at rest, in transit to the instance, and snapshots. Encrypt an unencrypted volume by copying its snapshot with encryption on.
- Snapshot extras: **Fast Snapshot Restore**, **Snapshot Archive** (cheaper, 24–72 h restore), **Recycle Bin** for accidental deletes. Automate with **Data Lifecycle Manager**.

### EC2 Instance Store

- Disks physically attached to the host. **Highest IOPS**, but **ephemeral**: data is lost if the instance stops, terminates or the hardware fails.
- Use for caches, buffers, scratch data, or replicated data (e.g. a distributed database).

!!! tip "Exam tip"
    "Highest possible disk performance, data can be lost" → **instance store**. "Data must survive stop/start" → **EBS**. "Need sustained high IOPS for a database" → **io2**.

---

## File storage

### Amazon EFS (Elastic File System)

- Managed **NFS** (v4.1) for **Linux**; mounted by **thousands** of EC2, ECS, EKS and Lambda clients across AZs. Grows and shrinks automatically, pay per GB used.
- **Storage classes**: Standard, Infrequent Access, Archive, moved by **lifecycle policies**. **One Zone** options are cheaper but single-AZ.
- **Throughput modes**: Elastic (recommended), Provisioned, Bursting. Encryption with KMS; access controlled by security groups and IAM/access points.

### Amazon FSx for Windows File Server

- Fully managed **SMB** file shares on Windows Server, with **Active Directory** integration, NTFS permissions and DFS namespaces.
- Single-AZ or **Multi-AZ**. Can be mounted from Linux too. Back up to S3.

### Amazon FSx for Lustre

- Parallel file system for **HPC, ML training, video processing and financial modelling**: hundreds of GB/s, millions of IOPS, sub-millisecond latency.
- Links to **S3**: reads S3 objects as files and can write results back.
- **Scratch** (temporary, cheapest, no replication) vs **Persistent** (replicated within an AZ, long-term).

!!! note "Other FSx options"
    **FSx for NetApp ONTAP** (NFS, SMB and iSCSI, good for migrating NetApp) and **FSx for OpenZFS** (migrating ZFS/Linux NFS) also exist.

!!! tip "Exam tip"
    "Shared file system for Linux instances" → **EFS**. "Windows, SMB, Active Directory" → **FSx for Windows**. "HPC / ML, high throughput, data in S3" → **FSx for Lustre**.

---

## Object storage

### Amazon S3 (Simple Storage Service)

- Buckets (globally unique names, created in a Region) hold objects up to **5 TB** each. Use **multipart upload** above 100 MB (required above 5 GB).
- **11 nines (99.999999999%) durability**; strong read-after-write consistency.
- Encryption at rest is **on by default** (SSE-S3); also SSE-KMS, DSSE-KMS or SSE-C. **Block Public Access** is on by default.
- Access via IAM policies, **bucket policies** (including cross-account), and access points.

### S3 storage classes

| Class | Access pattern | Min. storage duration | Retrieval |
|---|---|---|---|
| **Standard** | Frequent | None | Milliseconds |
| **Intelligent-Tiering** | Unknown or changing | None | Milliseconds (moves objects between tiers automatically) |
| **Standard-IA** | Infrequent, needs fast access | 30 days | Milliseconds, per-GB retrieval fee |
| **One Zone-IA** | Infrequent, re-creatable data | 30 days | Milliseconds; **single AZ** |
| **Glacier Instant Retrieval** | Accessed about once a quarter | 90 days | Milliseconds |
| **Glacier Flexible Retrieval** | Archive, rarely accessed | 90 days | Expedited 1–5 min, Standard 3–5 h, Bulk 5–12 h |
| **Glacier Deep Archive** | Long-term archive (compliance) | 180 days | Standard 12 h, Bulk up to 48 h |
| **Express One Zone** | Latency-sensitive, very high request rates | None | Single-digit milliseconds; single AZ |

- **Lifecycle rules** move objects to cheaper classes or expire them after N days.

### S3 versioning

- Keeps every version of an object; protects against overwrites and accidental deletes. Set at **bucket level**: Enabled or Suspended (never fully off once enabled).
- A delete adds a **delete marker**; remove the marker to restore the object.
- Required for **replication** (CRR/SRR) and **Object Lock**. **MFA Delete** adds protection for permanently deleting versions.
- Old versions cost money; clean them up with lifecycle rules (noncurrent version expiration).

### S3 access points

- Named endpoints on a bucket, **each with its own access policy**, so one big bucket policy becomes one policy per app/team.
- Can be limited to a **VPC only**. Variants: **Multi-Region Access Points** (one global endpoint, routes to the nearest bucket) and **Object Lambda Access Points** (transform data as it's read, e.g. redact PII).

!!! tip "Exam tip"
    "Unknown or changing access pattern" → **Intelligent-Tiering**. "Archive, retrieve within minutes rarely" → **Glacier Flexible Retrieval**. "Cheapest, 12-hour retrieval is fine" → **Deep Archive**. "Protect against accidental deletion" → **versioning (+ MFA Delete / Object Lock)**.

---

## Backup, DR & hybrid

### AWS Backup

- **Backup plans** (schedule, retention, lifecycle to cold storage) applied to resources by tag, across EC2, EBS, RDS, Aurora, DynamoDB, EFS, FSx, S3, Storage Gateway and more.
- Cross-Region and **cross-account** copies; works with AWS Organizations.
- **Backup Vault Lock**: WORM protection so backups can't be deleted, even by root.

### AWS Elastic Disaster Recovery (DRS)

- Continuous block-level replication of physical, virtual or cloud servers into a low-cost staging area in AWS.
- **RPO in seconds, RTO in minutes**; launch recovery instances on demand, run non-disruptive **drills**, then fail back.

### AWS Storage Gateway

| Gateway type | Protocol | Backed by | Use for |
|---|---|---|---|
| **S3 File Gateway** | NFS / SMB | S3 (objects) | File shares on-prem with data in S3; local cache for recent files |
| **FSx File Gateway** | SMB | FSx for Windows | Low-latency local access to FSx shares (no longer offered to new customers) |
| **Volume Gateway** | iSCSI | S3 + EBS snapshots | **Cached** (primary data in AWS) or **Stored** (primary data on-prem, async backup) volumes |
| **Tape Gateway** | iSCSI VTL | S3 / Glacier | Replace physical tapes with your existing backup software |

- Runs as a VM on-prem (VMware, Hyper-V, KVM) or as a hardware appliance.

---

## Quick decision cheat sheet

| Scenario | Answer |
|---|---|
| Boot/data disk for one EC2 instance | EBS gp3 |
| Database needing sustained high IOPS | EBS io2 Block Express |
| Big sequential throughput, low cost | EBS st1 |
| Fastest temporary scratch space | Instance store |
| Move an EBS volume to another AZ | Snapshot → restore in the new AZ |
| Shared file system for many Linux instances | EFS |
| Windows file shares with Active Directory | FSx for Windows File Server |
| HPC/ML with data in S3 | FSx for Lustre |
| Unknown access pattern, avoid retrieval fees | S3 Intelligent-Tiering |
| Infrequent access, re-creatable data, cheapest | S3 One Zone-IA |
| Archive, accessed once a quarter, instant retrieval | S3 Glacier Instant Retrieval |
| Compliance archive, cheapest, hours to restore | S3 Glacier Deep Archive |
| Recover overwritten/deleted objects | S3 versioning |
| Separate permissions per app on a shared bucket | S3 access points |
| Centralised backups across services and accounts | AWS Backup |
| Backups nobody can delete | AWS Backup Vault Lock / S3 Object Lock |
| Server DR with RPO seconds, RTO minutes | Elastic Disaster Recovery |
| On-prem NFS/SMB share backed by S3 | S3 File Gateway |
| Replace a physical tape library | Tape Gateway |
| On-prem iSCSI volumes backed up to AWS | Volume Gateway |
