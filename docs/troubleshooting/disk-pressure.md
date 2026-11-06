# Kubernetes Node DiskPressure Investigation

**Type:** Generalized incident pattern based on operational debugging.

## Signals
Pods evict or reschedule, a node reports `DiskPressure`, and node-level storage usage rises unexpectedly. Verbose application or sidecar logging to a hostPath can be one cause.

## Diagnostic commands
```bash
kubectl describe node NODE_NAME
kubectl get events -A --sort-by=.lastTimestamp
kubectl describe pod -n demo POD_NAME
kubectl top node NODE_NAME
```

On an authorized node, investigate filesystem capacity and large directories using approved access methods. `kubectl top` shows CPU/memory, **not disk usage**.

## Corrective actions
- Reduce excessive debug logging and apply retention/rotation.
- Prefer stdout/stderr with managed collection when suitable.
- Avoid unbounded hostPath writes.
- Set ephemeral-storage requests/limits where appropriate.
- Alert on node filesystem availability and growth.

## Prevention
Capture node storage signals alongside pod restart and eviction events; review sidecar logging behavior during application onboarding.
