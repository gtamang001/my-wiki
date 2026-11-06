# Lab: Namespace Isolation with Cilium

**Type:** Learning lab. Run only on a disposable cluster with Cilium network policy enforcement enabled.

## Goal
Allow intra-namespace traffic while denying other ingress to pods in a test namespace. This is **not** a complete production policy.

Create a namespace and a test server:
```bash
kubectl create namespace policy-lab
kubectl -n policy-lab create deployment web --image=nginx:stable
kubectl -n policy-lab expose deployment web --port=80
kubectl -n policy-lab rollout status deployment/web
```

Apply `examples/cilium/same-namespace-ingress.yaml` from the repository:
```bash
kubectl apply -f examples/cilium/same-namespace-ingress.yaml
kubectl -n policy-lab get ciliumnetworkpolicy
```

## Validate
Use temporary test pods in the same namespace and another namespace. Verify that same-namespace HTTP works and cross-namespace ingress is blocked. If a test client needs DNS resolution and has egress restrictions, allow CoreDNS explicitly.

## Cleanup
```bash
kubectl delete namespace policy-lab
```

!!! danger "Scope"
    This lab applies only to pods in `policy-lab`. Never apply a blanket clusterwide policy to production without a dependency inventory, staged rollout, exceptions, and rollback.
