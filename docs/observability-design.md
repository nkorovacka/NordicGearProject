# Observability

This covers how NordicGear sees what's happening across the platform, and how incidents
get noticed and investigated. It directly answers their ask for "a single place to see
metrics, logs, and maybe traces."

## The three types of data

- **Metrics** — numbers over time, like CPU usage, request counts, error rates. Good
  for spotting *that* something's wrong.
- **Logs** — the actual messages an app writes out, like "order #123 failed: payment
  declined." Good for finding out *why* something's wrong.
- **Traces** — follows one request as it moves between services, showing how long each
  step took. Good for finding *where* in the chain something is slow.

## Where each one goes

The EKS apps (frontend/backend/worker) send metrics to **Prometheus**, logs to **Loki**,
and traces to **AWS X-Ray**. Separately, AWS infrastructure itself (the ALB, RDS, NAT,
the EKS control plane) reports its own metrics and logs to **CloudWatch**, since that
part is managed by AWS rather than running inside the cluster.

All four of those feed into **Grafana**, which is the single dashboard NordicGear asked
for — instead of checking four different tools, the ops team looks in one place.

## How incidents are detected and investigated

Grafana has alert rules on top of the metrics ( like a high CPU, or
a pod stuck in a crash loop). When one triggers, it notifies the on-call engineer.

From there, the engineer investigates directly in Grafana - starting from the metric
that triggered the alert, then drilling into the related logs and traces to find the
actual root cause, all without switching tools.

## Diagram

![Observability](../diagrams/observability-diagram.svg)
