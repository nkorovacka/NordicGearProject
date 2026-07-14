# NordicGear cloud platform design
This is my design for NordicGear's new cloud platform. NordicGear is an online store
that sells outdoor gear in Europe. Right now their setup is messy (manual deployments,
missing logs, no real separation between environments), and this project is my plan for
a cleaner, more modern platform. This is just the design — I'm not building it, only
planning it out.

## Where to find things

| What | Where |
|---|---|
| Networking (VPC, subnets, how AZs work) | `docs/networking-design.md` |
| Kubernetes setup (namespaces, apps) | `docs/kubernetes-design.md` |
| How deployments work (CI/CD, GitOps) | `docs/CI-CD-design.md` |
| Security and access | `docs/security-design.md` |
| Logs, metrics, monitoring | `docs/observability-design.md` |
| Keeping costs under control | `docs/cost-design.md` |

## What this design looks like

When a customer visits the NordicGear website, their request goes
through a few safety and routing steps before it reaches the actual application. First it
goes through Route 53 (this is just AWS's DNS service — it points nordicgear.com to
the right place). Then it passes through AWS WAF, which checks the request isn't
something malicious like a bot attack, before it reaches the Load Balancer, which
spreads traffic across the app.

The app itself runs inside Kubernetes (EKS), split into three parts: a frontend
(what customers actually see), a backend (handles things like checkout and the
product catalog), and a worker (does background tasks, like sending confirmation
emails). The backend saves data into a database (RDS), stores files like product
images in S3, and puts messages in a queue (SQS) when something needs to happen
later. The worker picks up those queue messages and does the background job.

I'm building this same setup three times — for dev, staging, and prod — just
smaller for dev/staging and bigger (spread across multiple data centers, called
Availability Zones) for prod. That way, what works in staging should also work in prod,
since they're built the same way.

<img width="1041" height="946" alt="image" src="https://github.com/user-attachments/assets/7408b580-1a72-4541-a4b1-974f021b39ee" />

This shows the flow described above: users go through DNS and WAF into the load
balancer, which sends traffic into the Kubernetes cluster, where the three services
(frontend/backend/worker) talk to the database, file storage, and queue. The same setup
repeats, smaller, for staging and dev.
