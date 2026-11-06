# Kubernetes Operational Checklist

**Type:** Quick-reference troubleshooting checklist.

## Workload
```bash
kubectl get pods -A -o wide
kubectl describe pod -n demo POD_NAME
kubectl logs -n demo POD_NAME --previous
kubectl get events -A --sort-by=.lastTimestamp
```

## Service and ingress
```bash
kubectl get svc -A
kubectl get endpointslices -A
kubectl get ingress -A
kubectl get gateway,httproute -A
```

## Policy and cluster health
```bash
kubectl get networkpolicy -A
kubectl get ciliumnetworkpolicy -A
kubectl get nodes -o wide
kubectl top nodes
```

Always confirm context and namespace before changes:
```bash
kubectl config current-context
kubectl config view --minify
```

Prefer read-only diagnosis first. Capture the failing condition, timestamp, and expected behavior before modifying resources.
