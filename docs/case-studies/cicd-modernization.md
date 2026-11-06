# Enterprise CI/CD Modernization

**Type:** Production experience; generic patterns, no proprietary workflows.

## Context
Large repository estates benefit from standardized workflows, centralized security checks, and consistent deployment conventions. My work included migrating 500+ repositories to enterprise GitHub, using GitHub Actions at scale, and supporting 100+ Argo CD-backed repositories.

## Reference delivery flow
```mermaid
flowchart LR
  PR[Pull request] --> CI[Reusable GitHub Actions workflow]
  CI --> SEC[Tests and security checks]
  SEC --> ART[Versioned artifact]
  ART --> GIT[Deployment config update]
  GIT --> ARGO[Argo CD reconciliation]
  ARGO --> K8S[Kubernetes]
```

## Design principles
- Reusable workflows reduce duplicated pipeline logic.
- Keep build artifacts immutable and promotion auditable.
- Separate CI build/test responsibilities from GitOps reconciliation.
- Use least-privilege tokens, branch protection, code scanning, and approvals.
- Make rollback and drift detection visible to application teams.

## Trade-offs
A shared workflow improves consistency but can become a central dependency. Version workflows, provide upgrade guidance, and test changes before broad adoption.

## What to show in a public repository
An independently written reusable workflow, a sample container app, Helm values for multiple environments, and a minimal Argo CD Application manifest. Do not copy employer pipelines.
