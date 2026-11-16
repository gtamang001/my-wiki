---
description: Engineering case studies covering problem, approach, decisions and lessons learned.
---

# Case Studies

Real problems, retold: what went wrong (or needed to change), what I tried, the trade-offs, and what I'd tell my past self. Each one is labelled with its **type** so you know whether it ran in production or was a proof of concept.

| Case study | Type | Topics |
| --- | --- | --- |
| [Kubernetes network isolation with Cilium](cilium-isolation.md) | Production experience | Kubernetes, Cilium, security |
| [Gateway API: accepted is not reachable](gateway-api.md) | Evaluation / POC | Gateway API, Envoy, networking |
| [Enterprise CI/CD modernization](cicd-modernization.md) | Production experience | GitHub Actions, Argo CD, GitOps |
| [Observability and incident response](observability.md) | Production experience | Prometheus, Grafana, Loki |

!!! note "Public-safe by design"
    These stories use generic patterns and examples I wrote myself. No employer topology, manifests, pipelines or identifiers appear here, just the lessons.
