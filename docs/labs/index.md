---
description: Hands-on, reproducible labs for disposable Kubernetes and cloud environments.
---

# Labs

My playground. These are hands-on experiments where breaking things is encouraged, as long as it happens on a **disposable** cluster or account and never on anything someone depends on. :test_tube:

| Lab | What it shows | Environment |
| --- | --- | --- |
| [Namespace isolation with Cilium](cilium-policy.md) | Same-namespace allow, other ingress denied | Any cluster with Cilium |
| [Istio Ambient egress](ambient-egress.md) | What Ambient enrollment does and doesn't block | Cluster with Istio Ambient |
| [EKS networking & storage](eks-networking-storage.md) | VPC CNI, EBS vs EFS vs S3 | AWS account |

Example manifests live in the [`examples/`](https://github.com/gtamang001/my-wiki/tree/main/examples) folder of the repository.
