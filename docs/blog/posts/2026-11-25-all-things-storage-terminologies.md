---
date: 2026-11-25
authors:
  - ganga
categories:
  - Learning Log
tags:
  - Storage
---

# All things storage: a beginner's cheat sheet

Storage people speak their own language: LUNs, IOPS, erasure coding, thin provisioning… I collected the terms that finally made the storage landscape click for me, and boiled each one down to a single plain-English line.

<!-- more -->

No prior knowledge needed. Read the first section, and everything else will make much more sense.

## Start here: three ways to store data

Almost every storage product is one of these three. Think of a **parking garage**:

| Type | Plain English | Everyday picture | Examples |
|---|---|---|---|
| **Block** | A raw, empty disk. The computer formats it and decides how to organise it. | An empty parking space: park however you like. | A laptop's SSD, AWS EBS, iSCSI disks |
| **File** | Shared folders and files you open over the network. The storage server keeps them organised. | A valet who parks your car and remembers where. | Office shared drive, NFS, SMB, AWS EFS |
| **Object** | Files stored as "objects" with a label and a unique ID, fetched over the web. No folders, endless space. | A coat check: hand it over, get a ticket, show the ticket to get it back. | Amazon S3, MinIO, photo backups |

!!! tip "Rule of thumb"
    Databases and operating systems want **block**. Teams sharing documents want **file**. Huge amounts of photos, videos, backups and logs go to **object**.

## Where the storage lives

| Term | Plain English |
|---|---|
| **DAS** (Direct-Attached Storage) | A disk plugged straight into one computer. Simple, but not shared. |
| **NAS** (Network-Attached Storage) | A box on the network that shares **files** with many computers. |
| **SAN** (Storage Area Network) | A fast, dedicated network that gives servers **block** disks from a big central storage array. |
| **SDS** (Software-Defined Storage) | Storage run by software on ordinary servers instead of special hardware (e.g. Ceph, vSAN). |

## The disks themselves

| Term | Plain English |
|---|---|
| **HDD** | Spinning magnetic disk. Cheap and big, but slow. |
| **SSD** | Flash memory, no moving parts. Much faster, costs more per GB. |
| **NVMe** | A faster way of connecting SSDs, directly over PCIe. Today's speed king. |

## How computers talk to storage (protocols)

| Protocol | Type | Plain English |
|---|---|---|
| **iSCSI** | Block | Send disk commands over a normal network. The budget-friendly SAN. |
| **Fibre Channel (FC)** | Block | A dedicated, very reliable storage network. The classic enterprise SAN. |
| **FCoE** | Block | Fibre Channel traffic carried over Ethernet cables. |
| **NVMe-oF** | Block | NVMe speed over a network (Ethernet, TCP or Fibre Channel). |
| **NFS** | File | The standard way to share files with Linux/Unix machines. |
| **SMB / CIFS** | File | The standard way to share files with Windows. CIFS is the old version. |
| **S3 API** | Object | The common "language" for object storage, used well beyond AWS. |

## Carving up and organising disks

| Term | Plain English | Everyday picture |
|---|---|---|
| **File system** | The system that turns a raw disk into files and folders (ext4, XFS, NTFS). | The index and shelves in a library. |
| **LUN** | A numbered slice of a SAN handed to one server as "its disk". | A numbered storage unit in a big warehouse. |
| **Thick provisioning** | Reserve all the space up front. | Booking the whole restaurant in advance. |
| **Thin provisioning** | Promise the space, but only use it as data arrives. | An airline that overbooks, betting not everyone shows up. |
| **LVM** | Linux tool that pools disks together and lets you resize volumes easily. | Lego: join bricks and rebuild the shape. |
| **Multipathing (MPIO)** | Several cables/routes from server to storage, so one can fail and traffic keeps flowing. | Two roads to the same town. |
| **CSI** | The standard plug that lets Kubernetes use any storage system. | A universal power adapter. |
| **PV / PVC / StorageClass** | Kubernetes terms: a **PV** is a disk, a **PVC** is an app's request for one, a **StorageClass** is the menu of disk types. | Menu (class), order (PVC), the dish (PV). |

## Smart tricks file systems use

| Term | Plain English |
|---|---|
| **Copy-on-Write (CoW)** | Never overwrite data; write the change somewhere new and update the pointer. Makes snapshots instant and protects against corruption. |
| **Snapshot** | A point-in-time picture of a disk you can roll back to. Quick, but usually stored on the same system. |
| **ZFS / Btrfs** | Advanced file systems with built-in snapshots, checksums, compression and RAID. |
| **Checksum** | A fingerprint of data used to detect silent corruption ("bit rot"). |
| **Clustered file system** | One file system used by several servers at once, with locks so they don't clash (VMFS, GFS2, OCFS2). |
| **Deduplication** | Keep one copy of identical data. Like a group chat storing a shared photo once instead of for every member. |
| **Compression** | Shrink data to take less space, like zipping a file. |

## Speed: how fast is it?

Picture a **delivery company**:

| Term | Plain English | Delivery picture |
|---|---|---|
| **IOPS** | Read/write operations per second. Matters for databases (lots of small requests). | Deliveries per second |
| **Throughput** | Amount of data moved per second (MB/s, GB/s). Matters for video and backups (big files). | Total boxes moved per second |
| **Latency** | How long one request takes (milliseconds or microseconds). Lower is better. | How long one package takes to arrive |
| **Caching** | Keep frequently used data on faster storage (RAM, SSD). | A corner shop stocking the popular items. |
| **Tiering (hot / warm / cold)** | Move data to cheaper, slower storage as it gets used less. | Kitchen counter → cupboard → attic. |

## Keeping data safe

| Term | Plain English |
|---|---|
| **RAID** | Combine several disks so you get more speed, survive a disk failure, or both. **RAID 0**: fast, no protection. **RAID 1**: mirror, two identical copies. **RAID 5 / 6**: survives 1 / 2 disk failures using parity. **RAID 10**: mirror + stripe, fast and safe. |
| **Parity** | Extra calculated data that lets you rebuild a missing piece. Like a sudoku: with enough clues you can fill in the gap. |
| **Erasure coding** | RAID's modern big brother: split data into pieces plus parity pieces and spread them across many servers or sites. Used by cloud object storage. |
| **Replication (sync)** | Write to two places at the same moment. No data loss, but slower. |
| **Replication (async)** | Write locally, copy to the second place shortly after. Faster, but you could lose the last few seconds. |
| **Backup** | A separate copy you can restore from, ideally somewhere else. Snapshots and replication are **not** backups on their own (delete the original and the copy may go too). |
| **3-2-1 rule** | Keep **3** copies of data, on **2** different types of storage, with **1** off-site. |
| **Immutability / WORM** | "Write once, read many": data that can't be changed or deleted for a set time. The best defence against ransomware. |
| **Encryption at rest / in transit** | Scramble data while stored on disk / while it travels over the network. |
| **Durability vs availability** | **Durability**: will my data still exist? **Availability**: can I reach it right now? |
| **RPO** (Recovery Point Objective) | How much data you can afford to lose, measured in time ("up to 5 minutes' worth"). |
| **RTO** (Recovery Time Objective) | How long you can afford to be down ("back within 1 hour"). |

## If you remember only five things

1. **Block** = raw disk, **file** = shared folders, **object** = labelled items fetched by ID over the web.
2. **IOPS** (how many), **throughput** (how much), **latency** (how fast) describe performance.
3. **RAID and erasure coding** protect against failed disks; they're not backups.
4. Follow the **3-2-1 rule**, and keep one copy **immutable**.
5. **RPO** = how much data you can lose; **RTO** = how long you can be down.

:material-arrow-right: Want to see these ideas in the cloud? My [AWS storage notes](../../notes/aws/storage-services.md) map them to EBS (block), EFS and FSx (file) and S3 (object).
