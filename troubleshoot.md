# 🧠 PART 1 — Troubleshooting Mindset (Most Important)

**Before touching the CLI, always ask:**

1. **What is broken?**
2. **What changed?**
3. **When did it start?**
4. **Scope:** One pod / one namespace / whole cluster?

> **Never start blindly running commands.**

### 🧭 Universal Troubleshooting Flow

```sh
oc get → oc describe → oc logs → events → resource status
```

In 80% of cases, the problem is visible in:

- **Status field**
- **Events section**
- **Pod conditions**

---

# 🔵 PART 2 — Application Level Troubleshooting

## 🚨 Scenario 1 — Pod Not Running

**Check:**

```sh
oc get pods -n <ns>
```

If status is:

- `Pending`
- `CrashLoopBackOff`
- `Error`
- `ImagePullBackOff`

Each has a different cause.

---

### 🟡 Case A — Pod Pending

**Run:**

```sh
oc describe pod <pod> -n <ns>
```

**Look at:**

- **Events**

**Common causes:**

- ❌ Insufficient CPU/Memory  
    **Fix:** Reduce resource request, add nodes, check quotas
- ❌ PVC Not Bound  
    **Fix:**  
        ```sh
        oc get pvc
        ```
        Check StorageClass exists.
- ❌ NodeSelector mismatch  
    **Fix:** Fix labels or affinity.
- ❌ Taint without toleration  
    **Fix:** Add toleration or remove taint.

---

### 🔴 Case B — CrashLoopBackOff

**Run:**

```sh
oc logs <pod> -n <ns>
```

**Check:**

- App crash
- Wrong config
- Missing env var
- DB connection failure

**Fix:**

- Correct configmap
- Correct secret
- Check readiness/liveness probe

---

### 🟠 Case C — ImagePullBackOff

**Check:**

```sh
oc describe pod <pod>
```

**Common causes:**

- Wrong image name
- Private registry auth issue
- Network issue

**Fix:**

- Create imagePullSecret
- Check registry route
- Correct image tag

---

# 🔵 PART 3 — Deployment Issues

**Deployment not updating**

**Check:**

```sh
oc get deploy
oc describe deploy <name>
```

- ReplicaSet created?
- Events?
- Strategy?
- Image tag changed?

**Rollout stuck**

```sh
oc rollout status deploy/<name>
```

**Fix:**

- Check readiness probe
- Check failing pod

---

# 🔵 PART 4 — Service & Route Issues

**Route not accessible**

**Check:**

```sh
oc get route
oc describe route <name>
```

- TLS termination type
- Hostname
- Service exists?
- Endpoint exists?

**Service has no endpoints**

```sh
oc get endpoints
```

**Cause:** Label mismatch  
**Fix:** Ensure service selector matches pod labels.

---

# 🔵 PART 5 — Network Troubleshooting

**Pod cannot reach another pod**

**Inside pod:**

```sh
oc rsh <pod>
curl service-name
```

If fails:

- Check networkpolicy
- Check service selector
- Check DNS

**Inside pod:**

```sh
nslookup service-name
```

If fails:

- Check:
        ```sh
        oc get pods -n openshift-dns
        ```
- NetworkPolicy blocking traffic
        ```sh
        oc get networkpolicy -A
        ```
- **Fix:** Add allow policy.

---

# 🔵 PART 6 — Storage Troubleshooting

**PVC Pending**

```sh
oc get pvc
oc describe pvc
```

- StorageClass exists?
- Provisioner working?
- CSI driver running?

**Volume mount error**

- Check pod events:
        - Access mode mismatch
        - RWO used by multiple pods

**Fix:** Use correct access mode.

---

# 🔵 PART 7 — Authentication Issues

**LDAP login fails**

```sh
oc get oauth cluster -o yaml
```

- BindDN correct?
- BaseDN correct?
- URL correct?
- CA cert trusted?

**User has no permission**

```sh
oc auth can-i create pods --as <user>
```

- Check rolebindings.
- **Fix:**
        ```sh
        oc adm policy add-role-to-user
        ```

---

# 🔵 PART 8 — Monitoring & Performance

**High CPU pod**

```sh
oc adm top pods
```

**Scale deployment:**

```sh
oc scale deploy app --replicas=5
```

**Node high CPU**

```sh
oc adm top nodes
```

- Check Daemonsets
- Logging
- Large pods

**Prometheus down**

```sh
oc get pods -n openshift-monitoring
```

- Check events and PVC.

---

# 🔵 PART 9 — Logging Troubleshooting

- **App logs:**  
        ```sh
        oc logs <pod>
        ```
- **Previous crash logs:**  
        ```sh
        oc logs <pod> --previous
        ```
- **Node logs:**  
        ```sh
        oc adm node-logs <node>
        ```

---

# 🔵 PART 10 — Control Plane Troubleshooting

**API Server Slow**

```sh
oc get clusteroperators
```

If any operator degraded: **Describe it.**

**Cluster Operator Degraded**

```sh
oc describe co <name>
```

**Fix based on message.**

**Common issues:**

- Image registry
- Ingress
- Authentication
- Machine config

**Node NotReady**

```sh
oc get nodes
oc describe node
```

- Kubelet status
- Disk pressure
- Memory pressure

---

# 🔵 PART 11 — Debugging Tools

**Debug Node**

```sh
oc debug node/<node>
```

- Access host:
        ```sh
        chroot /host
        ```

**Debug Pod**

```sh
oc debug pod/<pod>
```

**Events**

Most powerful command:

```sh
oc get events -A --sort-by=.metadata.creationTimestamp
```

---

# 🔥 Real World Failure Scenarios

## Scenario 1 — App suddenly 503

- Check Route
- Service endpoints
- Pod readiness
- Crash logs

## Scenario 2 — After OADP restore app broken

- PVC restored?
- Secret restored?
- Configmap correct?

## Scenario 3 — GitOps keeps reverting changes

- ArgoCD sync policy
- Auto-sync enabled?

## Scenario 4 — Pod stuck Terminating

- Finalizers
- Volume detach issue
- Force delete carefully.

---

# 🧠 Advanced Architect Thinking

When issue affects:

- **One pod** → application issue
- **One namespace** → configuration issue
- **All namespaces** → cluster issue
- **API slow** → control plane issue

**Always determine blast radius.**

---

# 🎯 Master Troubleshooting Checklist

```sh
oc get pods
oc describe pod
oc logs
oc get events
oc get svc + endpoints
oc get pvc
oc auth can-i
oc adm top
oc get co
oc describe node
```
