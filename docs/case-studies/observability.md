# Observability and Incident Response

**Type:** Production experience, generalized troubleshooting lessons.

## Stack
Prometheus, Grafana, Alertmanager, Loki, Grafana Alloy, OpenSearch, and infrastructure exporters provide complementary views of infrastructure and application behavior.

## Investigation pattern
1. Confirm the symptom and impact window.
2. Compare metrics with events and logs.
3. Identify whether the issue is application, node, network, or platform related.
4. Validate the suspected cause with a narrowly scoped test.
5. Record mitigation, prevention, and useful alerts.

## Example: memory metrics
Container working set, RSS, and total memory usage answer different questions. Avoid diagnosing an application memory leak from a single metric without considering cache and process behavior.

## Example: logging sidecar and node pressure
A container writing verbose logs to host storage can cause node disk pressure even when the application appears healthy. See [DiskPressure troubleshooting](../troubleshooting/disk-pressure.md).

## Operational lesson
Useful dashboards and alerts should point to an action. A dashboard with many graphs but no clear failure hypotheses can slow down incident response.
