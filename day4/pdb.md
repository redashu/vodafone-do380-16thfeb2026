# PDB (PodDisruptionBudget) Guide

## 1️⃣ What is PDB (Concept)

**PDB = PodDisruptionBudget**

It protects applications from voluntary disruptions.

It defines: "How many pods must remain available during voluntary disruptions?"

## 2️⃣ What is a Voluntary Disruption?

### Examples

- `oc adm drain node`
- Cluster upgrade
- MachineConfig update
- Manual pod eviction
- Cluster autoscaler scale-down

### PDB DOES NOT protect against

- Node crash
- Power failure
- Kernel panic
- Pod crash

Those are involuntary disruptions.

## 3️⃣ Why PDB is Important

**Without PDB:** If you drain a node with 3 replicas → all can be evicted at once → downtime.

**With PDB:** Kubernetes ensures minimum availability during disruptions.

## 4️⃣ Two Ways to Define PDB

You must choose ONE:

### Option A — minAvailable

Minimum number of pods that must remain running.

```yaml
minAvailable: 2
```

If 3 replicas exist: Only 1 pod can be disrupted.

### Option B — maxUnavailable

Maximum pods that can be disrupted.

```yaml
maxUnavailable: 1
```

If 5 replicas: Only 1 pod can go down at a time.

## 5️⃣ How Scheduler Uses PDB

During node drain:

1. API checks PDB
2. If eviction violates PDB → eviction blocked
3. Drain waits or fails

PDB enforces safe maintenance.

## 6️⃣ SIMPLE DEMO

### Step 1 — Create Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: pdb-demo
spec:
    replicas: 3
    selector:
        matchLabels:
            app: pdbdemo
    template:
        metadata:
            labels:
                app: pdbdemo
        spec:
            containers:
            - name: nginx
                image: registry.access.redhat.com/ubi9/nginx-120
```

Apply:

```bash
oc apply -f deployment.yaml
oc get pods
```

You should see 3 pods.

### Step 2 — Create PDB

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
    name: pdb-demo
spec:
    minAvailable: 2
    selector:
        matchLabels:
            app: pdbdemo
```

Apply:

```bash
oc apply -f pdb.yaml
oc get pdb
```

Output shows `ALLOWED DISRUPTIONS: 1` — meaning only 1 pod can be disrupted.

### Step 3 — Simulate Node Drain

```bash
oc get pods -o wide
oc adm drain <node-name> --ignore-daemonsets --delete-emptydir-data
```

You will observe:

- Only 1 pod evicted at a time
- Drain blocks if more eviction breaks PDB

This is PDB protection.

## 7️⃣ What Happens Internally

During eviction, Kubernetes checks: `Current healthy pods - eviction_count >= minAvailable ?`

If condition fails → eviction denied.

## 8️⃣ Real Production Scenarios

- 🔵 **Rolling upgrade protection** — Prevent all replicas going down
- 🔵 **Node maintenance safety** — Ensure minimum availability
- 🔵 **Stateful applications** — Databases with replica quorum
- 🔵 **HPA + PDB combination** — Avoid scaling conflict

## 9️⃣ Important Edge Cases

**Case 1:** Only 1 replica + PDB minAvailable:1

- Node drain → blocked forever
- Must scale up first

**Case 2:** Cluster too small

- If replicas spread badly → PDB prevents eviction → upgrade blocked

**Case 3:** Anti-affinity + PDB

- Powerful combination for HA

## 🔟 Production Pattern

```yaml
# For 3 replicas:
minAvailable: 2

# For 5 replicas:
maxUnavailable: 1
```

Never use 100% strict PDB in small clusters.

## 1️⃣1️⃣ Relationship with HPA

If HPA scales down: It respects PDB. PDB protects minimum availability even during scaling.

## 1️⃣2️⃣ PDB vs Anti-Affinity

| Feature | PDB | Anti-Affinity |
| --- | --- | --- |
| Protect during drain | ✅ | ❌ |
| Spread pods | ❌ | ✅ |
| HA design | Partial | Yes |
| Maintenance safety | Yes | No |

Both should be used together in production.

## 1️⃣3️⃣ Simple Mental Model

- **Anti-affinity** → spread
- **PDB** → protect

## 1️⃣4️⃣ Enterprise Design Example

3 replica app across 3 nodes:

- ✔ Pod Anti-Affinity (one per node)
- ✔ PDB minAvailable:2

Now:

- Node crash → app still available
- Node drain → only 1 pod evicted

This is production-grade HA.

## 1️⃣5️⃣ Common Mistakes

- ❌ Using PDB with 1 replica
- ❌ Using minAvailable equal to replicas
- ❌ Forgetting selector labels
- ❌ Blocking cluster upgrades accidentally

## Architect Insight

OpenShift platform components (like etcd, router, monitoring) use PDB internally. Cluster upgrades rely on PDB respecting availability guarantees.