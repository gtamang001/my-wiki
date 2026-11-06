# Kubernetes DNS Troubleshooting

**Type:** Troubleshooting reference and learning notes.

## Start from inside a pod
```bash
kubectl exec -n demo deploy/client -- cat /etc/resolv.conf
kubectl exec -n demo deploy/client -- nslookup kubernetes.default.svc.cluster.local
kubectl get svc -n kube-system kube-dns
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl get endpointslices -n kube-system -l kubernetes.io/service-name=kube-dns
```

`dig` provides detailed DNS answers when installed:
```bash
dig kubernetes.default.svc.cluster.local A
# Query a specific DNS server (replace with the actual CoreDNS Service IP)
dig @10.96.0.10 kubernetes.default.svc.cluster.local A
```

## Interpret the failure
| Observation | Likely investigation |
| --- | --- |
| NXDOMAIN | Wrong hostname, namespace, search domain, or missing Service |
| Timeout | DNS network path, policy, CoreDNS health, or upstream |
| Name resolves but HTTP fails | Service, EndpointSlice, route, NetworkPolicy, or application |
| Works from one namespace only | Namespace-specific egress rules or DNS configuration |

## Additional checks
```bash
kubectl logs -n kube-system deploy/coredns --tail=100
kubectl get networkpolicy -A
kubectl get ciliumnetworkpolicy -A
kubectl get ciliumclusterwidenetworkpolicy
```

Never assume DNS is the root cause just because an application reports a connection error. Separate resolution from transport and HTTP failures.
