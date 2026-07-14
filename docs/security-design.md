# Security and Access

This covers how apps talk to AWS without shared credentials, how people access the
platform, and how secrets are handled. It answers NordicGear's biggest security
complaints: over-permissive IAM roles, shared credentials, and no hard-coded secrets.

## How apps access AWS services (no shared credentials)

Instead of giving each app a static AWS access key, I use **IAM Roles for Service
Accounts (IRSA)**. Each app (frontend, backend, worker) gets its own Kubernetes service
account, which is linked to its own IAM role with only the permissions that specific app
needs. For example, the backend's role allows reading/writing to the specific S3 bucket
and RDS database it uses -nothing more.

This means there's no access key sitting in a config file anywhere. AWS issues short-
lived, automatically-rotating credentials behind the scenes whenever the app needs them.
If a pod gets compromised, the damage is limited to exactly what that one app was
allowed to do — not the whole AWS account.

## How human access is controlled

Developers don't get AWS admin access. Instead, people log in through AWS IAM Identity
Center and get assigned a role with only the permissions their job actually needs - for example, a developer might get read-only access to logs and metrics, but not the
ability to change infrastructure directly.

Access to the Kubernetes cluster itself works the same way: Kubernetes RBAC controls who
can do what inside each namespace. A developer might be able to view pods and logs in
their own app's namespace, but not touch another team's namespace or cluster-wide
settings.

Since deployments go through the GitOps pipeline, nobody needs direct write access to the
cluster at all for normal changes - a change just needs to be approved and merged in
Git.

## How secrets are handled

Secrets (database passwords, API keys) are stored in AWS Secrets Manager, not in
code or in the GitOps repo. Apps read secrets at runtime using their IAM role — the
actual secret value never gets written down anywhere in Git history.

All data is encrypted at rest (RDS, S3) and in transit (TLS between the ALB and the
apps, and between apps and RDS). This directly satisfies NordicGear's requirement that
customer data must be encrypted both ways with no hard-coded credentials anywhere.
