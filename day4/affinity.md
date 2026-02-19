# Important Clarification

Affinity / Anti-Affinity is NOT actually on nodes.

It is defined in Pods, and it influences where pods run.

There are TWO different concepts:

- **Node Affinity** → Pod chooses node characteristics
- **Pod Affinity** → Pod chooses other pods nearby
- **Pod Anti-Affinity** → Pod avoids other pods

Most confusion happens here.

## High-Level Concept

### Node Affinity

Pod says: "Schedule me on nodes having these labels."

This controls hardware / node selection.

**Example:**

- SSD nodes
- GPU nodes
- Dedicated nodes

### Pod Affinity

Pod says: "Schedule me near another pod."

**Used when:**

- Frontend near backend
- App near cache
- Reduce latency

### Pod Anti-Affinity

Pod says: "Do NOT schedule me with similar pods."

**Used for:**

- High Availability
- Spread replicas across nodes/zones

## Visual Understanding

### Node Affinity

```
Node-A (disk=ssd)      ← allowed
Node-B (disk=hdd)      ← not allowed
```

### Pod Affinity

```
Node-A:
    backend pod
    frontend pod   ← wants same node
```

### Pod Anti-Affinity

```
Node-A: app-1
Node-B: app-2
Node-C: app-3

Pods spread across nodes.
```

## Hard vs Soft Rules

Both support:

- **Required (Hard rule)** - `requiredDuringSchedulingIgnoredDuringExecution` - Must match or pod stays Pending
- **Preferred (Soft rule)** - `preferredDuringSchedulingIgnoredDuringExecution` - Scheduler tries but may ignore

## Real Production Use Cases

**Node Affinity**

- GPU workloads
- DB on high-memory nodes
- Infra nodes

**Pod Affinity**

- App + DB close together
- Cache locality

**Pod Anti-Affinity**

- Avoid same-node replicas
- HA deployments

## Simple Demo

**Goal:** Pods run only on labeled nodes and replicas spread across nodes

### Step 1 — Label Nodes

```bash
oc label node worker-1 disk=ssd
oc label node worker-2 disk=ssd
oc get nodes --show-labels | grep disk
```

### Step 2 — Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: affinity-demo
spec:
    replicas: 2
    selector:
        matchLabels:
            app: demo
    template:
        metadata:
            labels:
                app: demo
        spec:
            affinity:
                nodeAffinity:
                    requiredDuringSchedulingIgnoredDuringExecution:
                        nodeSelectorTerms:
                        - matchExpressions:
                            - key: disk
                                operator: In
                                values:
                                - ssd
                podAntiAffinity:
                    requiredDuringSchedulingIgnoredDuringExecution:
                    - labelSelector:
                            matchLabels:
                                app: demo
                        topologyKey: kubernetes.io/hostname
            containers:
            - name: nginx
                image: registry.access.redhat.com/ubi9/nginx-120
```

### Step 3 — Verify Scheduling

```bash
oc get pods -o wide
```

Expected: Pods only on SSD nodes, NOT on same node

## Topology Key

In anti-affinity: `topologyKey: kubernetes.io/hostname` means "do not place pods on same node."

Other examples:

- `topology.kubernetes.io/zone` → Spread across AZs

## Memory Rule

- **Node Affinity** → WHERE (hardware)
- **Pod Affinity** → WITH WHOM
- **Pod Anti-Affinity** → AWAY FROM WHOM

## Enterprise Pattern

Production apps typically use: NodeAffinity + PodAntiAffinity + Taints/Tolerations for isolation, HA, and predictable scheduling.

## Common Mistakes

- Using required anti-affinity in small clusters → pods Pending
- Missing labels
- Wrong topology key
- Mixing nodeSelector + affinity incorrectly

## Platform Architecture

OpenShift platform components use affinity and anti-affinity to guarantee HA automatically.
