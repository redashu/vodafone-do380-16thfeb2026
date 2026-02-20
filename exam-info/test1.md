# 🧪 DO380 MOCK EXAM (Syllabus-Aligned)

**Duration:** 4 Hours  
**Environment:** Single OpenShift cluster  
**User:** student  
**Privileges:** cluster-admin

---

## SECTION 1 — Authentication & Identity Management (20%)

### Task 1 — Configure LDAP Authentication

Configure the cluster to authenticate using LDAP.

**Requirements:**

- LDAP server details are located in `/home/student/ldap-config.txt`
- Identity Provider name: `corp-ldap`
- Bind DN and password provided in file
- Base DN: `dc=example,dc=com`
- Map LDAP groups

**After configuration:**

- Sync groups manually
- Verify user `alice` can authenticate
- Grant `cluster-admin` role only to group `ocp-admins`

### Task 2 — Configure OIDC Authentication

Configure OIDC identity provider.

**Requirements:**

- OIDC configuration file: `/home/student/oidc-info.txt`
- Name the provider `corp-oidc`
- Map `preferred_username` as identity
- Ensure new login users are created automatically
- Verify user `developer1` can authenticate.

---

## SECTION 2 — Backup, Restore & Migration with OADP (25%)

### Task 3 — Deploy OADP

Deploy OADP in namespace `openshift-adp`.

**Requirements:**

- Use object storage credentials from `/home/student/object-storage.yaml`
- DataProtectionApplication name: `app-backup`
- Configure `BackupStorageLocation`
- Ensure status shows `Available`

### Task 4 — Create Namespace Backup

In namespace `sales-app`:

- Create backup named `sales-backup`
- Include only namespace `sales-app`
- Use snapshot-based backup
- Verify backup status is `Completed`.

### Task 5 — Restore Application

- Delete deployment `sales-api`.
- Restore from backup `sales-backup`.
- Verify deployment is running and ready.

### Task 6 — Create Scheduled Backup

Create a schedule named `daily-sales-backup`:

- Backup namespace `sales-app`
- Run daily at 02:00
- Retain 7 backups
- Verify schedule exists.

---

## SECTION 3 — Cluster Partitioning & Pod Scheduling (15%)

### Task 7 — Dedicated Node Configuration

- Label node `worker-2` with `workload=database`
- Taint node to allow only pods tolerating key `dedicated=db` with effect `NoSchedule`
- Verify taint is applied.

### Task 8 — Deploy Dedicated Workload

Deploy a deployment named `db-app`:

- 2 replicas
- Image: `registry.access.redhat.com/ubi9/nginx-120`
- Must run only on node labeled `workload=database`
- Must tolerate the taint applied earlier
- Verify both pods are running on `worker-2`.

### Task 9 — Prevent Other Workloads

- Deploy a pod named `test-pod` without toleration.
- Verify that it does NOT schedule on `worker-2`.

---

## SECTION 4 — OpenShift GitOps (20%)

### Task 10 — Install GitOps Operator

- Install OpenShift GitOps operator in namespace `openshift-gitops`.
- Create ArgoCD instance named `cluster-gitops`.
- Verify ArgoCD server is running.

### Task 11 — Deploy Application Using GitOps

Deploy application using GitOps:

- Repository: <https://git.example.com/apps/inventory.git>
- Path: `overlays/prod`
- Target namespace: `inventory`
- Enable automated sync with self-heal

**Verify:**

- Application is Synced
- Application is Healthy

### Task 12 — Demonstrate Drift Correction

- Manually scale deployment `inventory-api` to 1 replica.
- Verify GitOps restores replica count to original state.

---

## SECTION 5 — OpenShift Monitoring (10%)

### Task 13 — Identify High CPU Usage

- Identify which pod in namespace `analytics` is consuming highest CPU.
- Scale deployment appropriately to resolve performance issue.
- Verify pods are stable.

### Task 14 — Create Alert Rule

Create a Prometheus alert rule:

- Alert name: `HighMemoryUsage`
- Trigger when pod memory usage exceeds threshold
- Namespace: `analytics`
- Verify rule is applied.

---

## SECTION 6 — OpenShift Logging (10%)

### Task 15 — Deploy Cluster Logging

Deploy OpenShift Logging stack:

- Use default configuration
- Namespace: `openshift-logging`
- Ensure Elasticsearch and Kibana pods are running

### Task 16 — Query Logs

- Retrieve logs for pod `analytics-api`.
- Filter for error messages.
- Verify logs are visible via CLI.

---

## SECTION 7 — Certificate Management (5%)

### Task 17 — Secure Application Route

In namespace `finance-app`:

- Deploy application `finance-api`
- Expose service
- Configure Route with edge TLS termination
- Use provided secret `finance-tls`
- Verify HTTPS route is accessible.