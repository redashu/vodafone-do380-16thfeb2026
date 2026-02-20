# 🧪 DO380 MOCK EXAM — VERSION 2 (ADVANCED SCENARIO STYLE)

**Duration:** 4 Hours  
**Environment:** Single OpenShift cluster  
**Role:** cluster-admin

## SCENARIO

Your organization is preparing its OpenShift cluster for enterprise rollout.
You must secure authentication, protect workloads, enforce workload isolation, implement GitOps deployment, and ensure monitoring and logging are operational.

## SECTION 1 — Identity & Access Hardening (20%)

### Task 1 — Replace Default Authentication

The cluster currently uses only kubeadmin.

You must:

- Configure LDAP authentication using details in `/home/student/ldap-prod.txt`
- Name IdP `enterprise-ldap`
- Only allow members of LDAP group `platform-admins` to have cluster-admin access
- Ensure non-admin LDAP users can log in but have no cluster privileges

Validate by testing login for:
- `adminuser`
- `developeruser`

### Task 2 — Enforce Namespace-Level Access

Create namespace `research`.

Requirements:
- Create role `research-editor`
- Allow create, update, delete on deployments only
- Bind role to LDAP group `research-team`
- Ensure members cannot delete services

Verify using user impersonation.

## SECTION 2 — Application Data Protection (OADP) (25%)

### Task 3 — Partial Namespace Backup

Namespace `billing` contains multiple resources.

Create a backup:
- Name: `billing-config-only`
- Include only ConfigMaps and Secrets
- Exclude PersistentVolumes
- Store in configured BackupStorageLocation

Verify completion.

### Task 4 — Snapshot-Based Backup

Namespace `payments` contains PVC-backed deployment.

Create backup `payments-snap`:
- Use CSI snapshot
- Ensure VolumeSnapshot is created
- Verify snapshot content exists

### Task 5 — Disaster Recovery Simulation

Namespace `payments` deployment `payment-api` has been deleted accidentally.

Restore only this deployment from `payments-snap` without restoring the full namespace.

Verify service becomes available.

## SECTION 3 — Workload Isolation & Scheduling Control (15%)

### Task 6 — Isolate Compute-Intensive Workloads

Node `worker-1` must be reserved for compute workloads.

- Label node appropriately
- Taint node to prevent non-compute workloads
- Deploy deployment `compute-app`:
    - Must run exclusively on `worker-1`
    - 3 replicas

Verify placement.

### Task 7 — Enforce High Availability

Deployment `frontend-app` must:
- Spread across different nodes
- Never schedule two replicas on same node
- Maintain minimum 2 pods during node drain

Implement required scheduling and protection configuration.

Verify using pod distribution.

## SECTION 4 — GitOps Operational Model (20%)

### Task 8 — Bootstrap GitOps for Multi-Environment

Install OpenShift GitOps.

Create:
- ArgoCD instance named `platform-gitops`
- Deploy two applications from:
    - Repo: <https://git.example.com/multi-env.git>
    - Applications:
        - `dev-app` (path: dev)
        - `prod-app` (path: prod)

Requirements:
- `dev-app` must sync manually
- `prod-app` must auto-sync with prune enabled

Verify sync status.

### Task 9 — Recover From Configuration Drift

A manual change was made:
- `prod-app` deployment replica count changed to 1.

Restore desired state using GitOps without editing cluster directly.

Verify replica count matches Git.

## SECTION 5 — Monitoring & Performance Troubleshooting (10%)

### Task 10 — Investigate Pod Crash

Namespace `orders` contains crashing pod `orders-api`.

- Identify root cause using monitoring tools.
- Apply configuration fix.
- Ensure pod becomes stable.

### Task 11 — Create Custom Monitoring Rule

Create PrometheusRule in namespace `orders`:
- Alert when pod restarts exceed threshold
- Name rule `HighRestartAlert`

Verify rule is active.

## SECTION 6 — Logging & Observability (10%)

### Task 12 — Deploy Cluster Logging With Custom Retention

Deploy cluster logging stack.

Configure:
- Log retention 3 days
- Namespace: `openshift-logging`

Verify Elasticsearch and collector pods are running.

### Task 13 — Troubleshoot Application Logs

Application `orders-api` produces errors.

Using logging stack:
- Query logs
- Identify error pattern
- Provide evidence via CLI

## SECTION 7 — Secure Application Exposure (5%)

### Task 14 — Enforce Encrypted External Access

Namespace `reports` contains service `reports-api`.

Create route:
- Termination type: re-encrypt
- Use provided secret `reports-cert`
- Disable insecure edge traffic

Verify route configuration.