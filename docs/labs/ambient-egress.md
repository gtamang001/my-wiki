# Lab Notes: Istio Ambient Egress

**Type:** Evaluation/POC and learning notes, not a claim of completed production rollout.

## Question
Does enrolling a namespace in Istio Ambient automatically block unknown external domains?

**No.** Ambient enrollment alone does not establish a universal hostname egress deny policy. L4 policy and waypoint/L7 policy have different capabilities, and the exact enforcement path matters.

## Investigation workflow
1. Verify Istio CNI and `ztunnel` are installed and healthy.
2. Confirm namespace enrollment and pod redirection.
3. Record the source workload identity and destination.
4. Verify waypoint association where L7 policies are expected.
5. Test HTTP and HTTPS separately; TLS passthrough and hostname inspection are not interchangeable.
6. Compare expected allow/deny behavior with observed status codes and telemetry.

```bash
kubectl get ns --show-labels
kubectl get ds -n istio-system
kubectl get pods -n istio-system
kubectl get authorizationpolicy -A
kubectl get serviceentry -A
```

## Lessons
- A `200` from an unexpected destination is evidence that the intended restriction did not apply.
- A `503` can reflect route/waypoint configuration rather than a deliberate deny.
- Policy attachment, export visibility, TLS semantics, and traffic interception must be validated individually.
- Consider complementary Cilium network policy for lower-layer isolation.
