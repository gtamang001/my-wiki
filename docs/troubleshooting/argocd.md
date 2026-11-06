# Argo CD Reconciliation Troubleshooting

**Type:** Operational troubleshooting reference.

## Symptoms
- Application is OutOfSync or Degraded.
- A Git repository cannot be fetched.
- A manual fix is reverted by the next reconciliation.
- Resources remain after application removal because ownership/finalizers differ.

## Checks
```bash
kubectl get applications.argoproj.io -A
kubectl get pods -n argocd
kubectl logs -n argocd deploy/argocd-repo-server --tail=100
kubectl get events -n argocd --sort-by=.lastTimestamp
```

## Investigation approach
1. Inspect application conditions and the exact failing resource.
2. Verify repo credentials, network reachability, and branch/path/revision.
3. Compare live state with the declared Git source.
4. Check Helm/Kustomize rendering and Kubernetes admission errors.
5. Fix the source of truth rather than repeatedly patching live resources.

!!! warning
    Argo CD and Helm may both claim ownership of resources. Before deleting or adopting objects, review labels, annotations, release metadata, and rollback plans.
