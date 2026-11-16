---
description: Troubleshooting runbooks indexed by symptom — Kubernetes, DNS, GitOps and node health.
---

# Troubleshooting

Every fix I make has a little story behind it. This is where I write them down, so the next time the pager goes off, the answer is already waiting. Each runbook follows the same path: **symptoms → checks → investigation → fix → prevention**.

## Find by symptom

| Symptom | Runbook |
| --- | --- |
| `NXDOMAIN`, DNS timeouts, or names resolve in one namespace only | [Kubernetes DNS](kubernetes-dns.md) |
| Argo CD app is `OutOfSync` / `Degraded`, or manual fixes get reverted | [Argo CD reconciliation](argocd.md) |
| Node reports `DiskPressure`, pods are evicted | [Node DiskPressure](disk-pressure.md) |
| Gateway is `Programmed` but clients get connection refused / 503 | [Gateway API case study](../case-studies/gateway-api.md) |

!!! tip "General approach"
    Look before you touch. Note the symptom, the time and what *should* be happening before changing anything. Then check DNS, the connection, TLS and HTTP one at a time. (Yes, it's often DNS. No, not always.) See the [Kubernetes operational checklist](../notes/kubernetes/operational-checklist.md) and [networking commands](../notes/networking/commands.md).
