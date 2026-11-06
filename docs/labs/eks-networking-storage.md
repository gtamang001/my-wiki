# EKS Networking and Storage — Learning Notes

**Type:** Learning lab and conceptual reference, not a claim of operating EKS in production.

## Networking
The default Amazon VPC CNI allocates pod IPs from VPC subnets using ENIs and secondary IPs or prefixes. Cilium-managed clusters may use different IPAM and routing models. Understand VPC subnet capacity, ENI/IP limits, security groups, and how pods reach Services and external networks.

## Storage choices
| Service | Typical use | Important constraint |
| --- | --- | --- |
| EBS | Single-AZ block volumes for stateful workloads | Volume is tied to an Availability Zone; attach/access rules apply |
| EFS | Shared NFS storage across multiple AZs | Network filesystem latency, throughput, and cost |
| S3 | Object storage for backups/artifacts | Not a normal POSIX-mounted Kubernetes volume by default |

## Provisioning checklist
- Select region, VPC, subnet and AZ design.
- Establish IAM roles and cluster/node permissions.
- Plan Kubernetes version and managed add-ons.
- Configure CNI, CSI drivers, autoscaling and observability.
- Test pod scheduling, volume attachment, cross-AZ behavior and teardown.

## Interview exercise
What happens if an EBS-backed pod is rescheduled to a different Availability Zone? Explain scheduling constraints, volume topology, and recovery choices.
