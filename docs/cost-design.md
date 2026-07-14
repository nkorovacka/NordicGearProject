# Cost Awareness

This covers how the design avoids paying for capacity that isn't being used, and how I'd
keep compute and observability costs under control - directly answering NordicGear's
concern about rising cloud costs, especially during peak periods.

## Scaling up and down automatically

Rather than running enough servers year-round to handle Black Friday traffic, the
cluster scales itself based on actual demand:

- **Horizontal Pod Autoscaler** adds more pods (frontend/backend/worker replicas) when
  CPU or request load goes up, and removes them again once traffic drops.
- **Cluster Autoscaler** (or Karpenter) adds more EKS worker nodes when there isn't
  enough room to schedule those extra pods, and removes nodes again once they're no
  longer needed.

This is what makes "scale up during sales and back down afterward" actually happen
automatically, instead of someone manually resizing things before and after every
campaign.

## Right-sizing each environment

Dev and staging don't need to be full-size copies of prod. They run smaller instance
types, fewer replicas, and a single AZ instead of two - enough to test that something
works, without paying for prod-level redundancy in an environment nobody actually
depends on staying up 24/7.

## Using cheaper compute where it's safe to

Worker nodes running background jobs (and most of dev/staging) can run on **Spot
Instances**, which cost significantly less than on-demand pricing. This is safe for
workloads that can tolerate being interrupted and restarted - which fits the worker
service well, since it's just processing a queue and can pick up where it left off.
Prod's steady baseline capacity can use **Savings Plans** to lower the cost of the
capacity that's always running, while still bursting to on-demand or spot for spikes.

## Keeping observability costs in check

Collecting metrics, logs, and traces isn't free - storing and querying all of it adds
up, especially at high traffic volumes. To keep this reasonable:

- Logs have a retention period (e.g. 30 days) instead of being kept forever.
- Traces are sampled (e.g. capturing a percentage of requests) rather than recording
  every single one, since a representative sample is usually enough to spot problems.
- Metrics stay at a reasonable resolution instead of tracking every possible detail at
  the finest possible granularity.

## Other small things that add up

S3 can automatically move older files (like old logs or infrequently accessed assets)
into cheaper storage tiers over time, instead of paying full price to keep everything in
the most expensive tier forever.
