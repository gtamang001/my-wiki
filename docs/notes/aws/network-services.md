---
description: AWS exam notes on networking, covering VPC, subnets, routing, internet and NAT gateways, DNS, Elastic IPs, security groups and NACLs, load balancers, VPN, Direct Connect, VPC peering, Transit Gateway, PrivateLink, CloudFront, Global Accelerator and Route 53.
tags:
  - AWS
  - Networking
  - Exam Prep
---

# AWS Network Services

Quick revision notes for the Solutions Architect exam. Networking questions are about **isolating** workloads in a VPC, **connecting** VPCs and on-prem networks, and **delivering** content to users quickly. The table below gives one line per service; the sections after it go deeper on the ones that show up most in questions.

## At a glance

| Service | Category | One-liner |
|---|---|---|
| **VPC** | Core | Your private, isolated network in a Region |
| **Subnets** | Core | A slice of the VPC's IP range in **one AZ** |
| **Route tables** | Core | Decide where traffic from a subnet goes |
| **Internet Gateway** | Core | Lets a VPC talk to the internet (both ways) |
| **NAT Gateway** | Core | Lets **private** subnets reach the internet, outbound only |
| **Public / private subnets** | Core | Public = route to an Internet Gateway; private = no direct route |
| **VPC DNS** | Core | Built-in Route 53 Resolver for name resolution inside the VPC |
| **Elastic IP** | Core | A static public IPv4 address you own until you release it |
| **Security groups & NACLs** | Security | Instance-level (stateful) and subnet-level (stateless) firewalls |
| **Elastic Load Balancing** | Traffic | Spreads traffic across targets: ALB (L7), NLB (L4), GWLB (L3) |
| **Site-to-Site / Client VPN** | Hybrid | Encrypted IPsec tunnels over the internet to your VPC |
| **Direct Connect** | Hybrid | Private, dedicated network link from your data centre to AWS |
| **VPC Peering** | Connectivity | Private link between two VPCs (not transitive) |
| **Transit Gateway** | Connectivity | Hub that connects many VPCs and on-prem networks (transitive) |
| **PrivateLink** | Connectivity | Private access to AWS services or another VPC's service, no internet |
| **CloudFront** | Edge | CDN that caches content at edge locations worldwide |
| **Lambda@Edge** | Edge | Run Lambda functions at CloudFront edge locations |
| **Global Accelerator** | Edge | Two static anycast IPs that route users over the AWS backbone |
| **Route 53** | DNS | Managed DNS, domain registration, health checks and routing policies |
| **Route 53 ARC** | Resilience | Readiness checks and routing controls for multi-Region/multi-AZ failover |

---

## VPC fundamentals

### VPC (Virtual Private Cloud)

- **Regional**, spans all AZs. IPv4 CIDR between **/16 and /28** (add more CIDRs later); optional IPv6.
- Each Region has a **default VPC** with public subnets, so instances get public IPs out of the box.
- Use private ranges (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16) and **don't overlap** with networks you'll connect to.

### Subnets

- Each subnet lives in **one AZ**. Spread subnets over 2+ AZs for high availability.
- AWS **reserves 5 IPs** per subnet (first four and the last), so a /24 has 251 usable addresses.

### Routing in a VPC

- Every subnet is associated with one **route table** (the main one if you don't choose). The **local** route lets everything in the VPC talk to each other.
- The **most specific route wins** (longest prefix match).
- Targets include IGW, NAT gateway, VPC peering, Transit Gateway, VGW and gateway endpoints.

### Internet Gateway (IGW)

- One per VPC, horizontally scaled and highly available. Allows inbound and outbound internet traffic for resources with public IPs.
- **Egress-only Internet Gateway**: outbound-only for **IPv6** (the IPv6 equivalent of NAT).

### NAT Gateway

- Lets instances in **private subnets** start connections to the internet; nothing from the internet can start a connection to them.
- Lives in a **public subnet** with an Elastic IP; managed, scales to 100 Gbps. It's **AZ-scoped**: create **one per AZ** for HA.
- **NAT instance** is the old, self-managed option (needs source/dest check disabled).

### Public and private subnets

- **Public subnet**: route table has `0.0.0.0/0 → IGW`; instances need a public or Elastic IP. Put load balancers, NAT gateways, bastions here.
- **Private subnet**: no route to the IGW; reaches the internet via NAT. Put app servers and databases here.

### DNS in a VPC

- **Route 53 Resolver** (the `.2` address of the VPC) resolves names. Controlled by `enableDnsSupport` and `enableDnsHostnames`.
- **Private hosted zones** give custom internal domain names.
- **Resolver endpoints** for hybrid DNS: **inbound** (on-prem resolves AWS names), **outbound** (AWS resolves on-prem names via forwarding rules).

### Elastic IP

- Static public IPv4 that you can remap to another instance or ENI to mask a failure.
- All public IPv4 addresses (including Elastic IPs) are **charged hourly**. Prefer a load balancer or DNS over juggling EIPs.

### Security groups vs NACLs

| | Security group | Network ACL |
|---|---|---|
| Applies to | Instance / ENI | Subnet |
| State | **Stateful** (return traffic allowed automatically) | **Stateless** (allow return traffic explicitly, incl. ephemeral ports 1024–65535) |
| Rules | **Allow only** | Allow **and deny** |
| Evaluation | All rules evaluated | Rules in **number order**, first match wins |
| Typical use | Main firewall; can reference other security groups | Block specific IPs/ranges at subnet level |

!!! tip "Exam tip"
    "Block a specific malicious IP" → **NACL deny rule** (security groups can't deny). "Allow only the app tier to reach the DB" → **DB security group referencing the app security group**.

---

## Load balancing

| Type | Layer | Highlights |
|---|---|---|
| **Application LB (ALB)** | 7 (HTTP/HTTPS, gRPC, WebSocket) | Routes by path, host, headers, query string; targets EC2, IP, Lambda, containers; WAF integration |
| **Network LB (NLB)** | 4 (TCP, UDP, TLS) | Millions of requests/sec, ultra-low latency, **static IP / Elastic IP per AZ**, PrivateLink |
| **Gateway LB (GWLB)** | 3 (IP, GENEVE port 6081) | Sends traffic through fleets of **third-party firewalls/IDS** transparently |
| **Classic LB** | 4/7 | Legacy, avoid for new designs |

- Health checks, cross-zone load balancing, TLS termination with **ACM** certificates, sticky sessions, connection draining (deregistration delay).

---

## Hybrid connectivity

### AWS VPN

- **Site-to-Site VPN**: IPsec tunnels between your **customer gateway** (on-prem device) and a **virtual private gateway** or **Transit Gateway**. Two tunnels for redundancy; up to about 1.25 Gbps per tunnel. Quick to set up, runs over the internet.
- **Client VPN**: managed OpenVPN-based access for individual users' laptops.
- **VPN CloudHub**: hub-and-spoke between multiple on-prem sites through one VGW.

### AWS Direct Connect (DX)

- Private, dedicated connection from your data centre to AWS via a DX location. **Consistent latency, high bandwidth** (dedicated 1/10/100/400 Gbps; hosted from 50 Mbps), lower data-transfer cost.
- Takes **weeks** to provision. **Not encrypted** by default: add **MACsec** or run a **VPN over DX**.
- **Direct Connect Gateway** connects one DX to VPCs in many Regions.
- Resilience: two connections at two DX locations, or **Site-to-Site VPN as backup**.

!!! tip "Exam tip"
    "Need connectivity quickly / cheaply" → **VPN**. "Consistent performance, large data volumes" → **Direct Connect**. "Encrypted private link" → **VPN over Direct Connect** or MACsec.

---

## VPC-to-VPC & private connectivity

### VPC Peering

- Private connection between two VPCs (same or different account/Region) over the AWS network.
- **Not transitive** (A–B and B–C doesn't give A–C), **no overlapping CIDRs**, routes must be added on both sides. Fine for a few VPCs; becomes a mesh at scale.

### Transit Gateway (TGW)

- Regional **hub** that connects thousands of VPCs, VPNs and Direct Connect. **Transitive** routing, controlled by TGW route tables.
- Inter-Region peering, share across accounts with **RAM**, and **ECMP** across multiple VPN tunnels to increase VPN bandwidth.

### AWS PrivateLink & VPC endpoints

- **Interface endpoint** (PrivateLink): an ENI with a private IP in your subnet for an AWS service (SQS, KMS, Secrets Manager…) or a partner/your own service behind an **NLB**. Paid per hour and per GB.
- **Gateway endpoint**: a route-table target for **S3 and DynamoDB only**; **free**.
- Lets you expose a service to thousands of VPCs without peering and without overlapping-CIDR problems.

!!! tip "Exam tip"
    "Private instances reach S3 without NAT or internet" → **S3 gateway endpoint**. "Share a service with many other VPCs/accounts" → **PrivateLink (NLB + endpoint service)**. "Connect many VPCs and on-prem" → **Transit Gateway**.

---

## Edge & DNS

### Amazon CloudFront

- CDN with hundreds of edge locations; caches static and dynamic content, reduces latency and origin load.
- Origins: S3 (protect with **Origin Access Control**), ALB, EC2, any HTTP server.
- **Signed URLs/cookies** for private content, **geo restriction**, HTTPS, WAF and Shield integration, field-level encryption.

### Lambda@Edge (and CloudFront Functions)

- **Lambda@Edge**: Node.js/Python functions on viewer or origin request/response; up to seconds of run time, network access. Good for auth, A/B testing, rewriting responses.
- **CloudFront Functions**: lightweight JavaScript for viewer request/response only, sub-millisecond, much cheaper. Good for header manipulation, URL rewrites, simple redirects.

### AWS Global Accelerator

- Gives you **two static anycast IPs**; users enter the AWS backbone at the nearest edge and are routed to the best healthy endpoint (ALB, NLB, EC2, EIP) in one or more Regions.
- Works for **TCP and UDP** (non-HTTP too), **fast regional failover**, no caching.

!!! tip "Exam tip"
    "Cache content close to users" → **CloudFront**. "Static IPs, non-HTTP (gaming, VoIP), fast multi-Region failover" → **Global Accelerator**.

### Amazon Route 53

- Highly available DNS, domain registrar and **health checks**. Public and private hosted zones.
- **Alias records**: point to AWS resources (ELB, CloudFront, S3 website) and work at the **zone apex**; free queries. CNAME can't be used at the apex.

| Routing policy | Use for |
|---|---|
| **Simple** | One resource |
| **Weighted** | Split traffic by percentage (blue/green, canary) |
| **Latency** | Send users to the Region with the lowest latency |
| **Failover** | Active-passive with health checks |
| **Geolocation** | Route by user's country/continent (content, compliance) |
| **Geoproximity** | Route by distance, with a bias to shift traffic |
| **Multivalue answer** | Return up to 8 healthy records (client-side load balancing) |
| **IP-based** | Route by the client's IP range |

### Route 53 Application Recovery Controller (ARC)

- **Readiness checks**: confirm your standby replica (capacity, quotas, config) is ready to take traffic.
- **Routing controls**: highly available on/off switches to fail traffic over between Regions or cells.
- **Zonal shift / zonal autoshift**: move load-balancer traffic away from an impaired AZ.

---

## Quick decision cheat sheet

| Scenario | Answer |
|---|---|
| Private instances need outbound internet | NAT gateway in a public subnet (one per AZ) |
| IPv6 outbound only | Egress-only Internet Gateway |
| Static public IP for an instance | Elastic IP |
| Block a specific IP range | NACL deny rule |
| Allow only the app tier to reach the database | Security group referencing the app SG |
| Route HTTP by path or host | ALB |
| Static IP, TCP/UDP, extreme performance | NLB |
| Insert third-party firewall appliances | Gateway Load Balancer |
| Quick, encrypted on-prem connection | Site-to-Site VPN |
| Consistent, high-bandwidth private link | Direct Connect (+ VPN backup) |
| One DX to VPCs in many Regions | Direct Connect Gateway |
| Connect two VPCs | VPC peering |
| Connect many VPCs and on-prem networks | Transit Gateway |
| Private access to S3/DynamoDB, free | Gateway endpoint |
| Private access to other AWS services | Interface endpoint (PrivateLink) |
| Expose a service to many VPCs privately | PrivateLink endpoint service (NLB) |
| Resolve on-prem names from AWS | Route 53 Resolver outbound endpoint |
| Cache static content globally | CloudFront |
| Lightweight header rewrites at the edge | CloudFront Functions |
| Auth / heavier logic at the edge | Lambda@Edge |
| Static anycast IPs, fast multi-Region failover | Global Accelerator |
| Point the apex domain at a load balancer | Route 53 alias record |
| Send users to the lowest-latency Region | Route 53 latency routing |
| Controlled multi-Region failover with readiness checks | Route 53 ARC |
