# 🧠 First: What are jq and yq?

| Tool | Works on | Primary use |
|------|----------|-------------|
| jq | JSON | Filter, delete, transform JSON |
| yq | YAML (and JSON) | Filter, delete, transform YAML |

👉 Kubernetes API speaks JSON internally
👉 Humans usually export YAML

That's why both tools are common in OCP work.

## 🔹 jq — JSON Processor (Very Powerful, Very Precise)

### When to use jq

- You export `-o json`
- You want precise field deletion
- You are scripting / automating
- You don't want extra dependencies

### 🔑 jq Basic Syntax

```bash
jq 'FILTER'
```

**Examples:**

**Pretty-print JSON**
```bash
jq .
```

**Delete a single field**
```bash
jq 'del(.status)'
```

**Delete multiple metadata fields (K8s cleanup)**
```bash
jq 'del(
    .metadata.uid,
    .metadata.resourceVersion,
    .metadata.creationTimestamp,
    .metadata.generation,
    .metadata.managedFields,
    .status
)'
```

### 🔥 jq — Clean a Deployment (Production Style)

```bash
oc get deployment webapp -o json \
| jq 'del(
        .metadata.uid,
        .metadata.resourceVersion,
        .metadata.creationTimestamp,
        .metadata.generation,
        .metadata.managedFields,
        .status
    )' > clean-deployment.json
```

### 🔥 jq — Clean ALL Objects in a Namespace

```bash
oc get all,cm,secret,pvc,route -n webapp -o json \
| jq 'del(
        .items[].metadata.uid,
        .items[].metadata.resourceVersion,
        .items[].metadata.creationTimestamp,
        .items[].metadata.generation,
        .items[].metadata.managedFields,
        .items[].status
    )' > webapp-clean.json
```

### ⚠ jq Limitations

- Output is JSON (not YAML)
- Harder to read for humans
- Needs yq if you want YAML output

## 🔹 yq — YAML Processor (Human-Friendly)

### When to use yq

- You work directly with YAML
- You want readable manifests
- You are editing/exporting manifests
- You don't want JSON at all

### 🔑 yq Versions (IMPORTANT)

There are two yq tools:

| Version | Command style |
|---------|---------------|
| yq v3 (old) | `yq d file.yaml field` |
| yq v4 (current, recommended) | `yq eval 'del(.field)'` |

Below assumes yq v4.

**Check version:**
```bash
yq --version
```

### 🔥 yq — Clean YAML Directly

**Remove status**
```bash
yq eval 'del(.status)' app.yaml
```

**Remove metadata garbage**
```bash
yq eval 'del(
    .metadata.uid,
    .metadata.resourceVersion,
    .metadata.creationTimestamp,
    .metadata.generation,
    .metadata.managedFields
)' app.yaml
```

### 🔥 yq — One-Liner Clean Export from Cluster

```bash
oc get deployment webapp -o yaml \
| yq eval 'del(
        .metadata.uid,
        .metadata.resourceVersion,
        .metadata.creationTimestamp,
        .metadata.generation,
        .metadata.managedFields,
        .status
    )' - > clean-deployment.yaml
```

### 🔥 yq — Clean List Objects (Namespace Backup)

```bash
oc get all,cm,pvc,route -n webapp -o yaml \
| yq eval 'del(
        .items[].metadata.uid,
        .items[].metadata.resourceVersion,
        .items[].metadata.creationTimestamp,
        .items[].metadata.generation,
        .items[].metadata.managedFields,
        .items[].status
    )' - > webapp-clean.yaml
```

## 🧠 jq vs yq — Direct Comparison

| Feature | jq | yq |
|---------|----|----|
| Input | JSON | YAML + JSON |
| Output | JSON | YAML |
| Precision | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Readability | ❌ | ✅ |
| Automation | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| K8s Cleanup | Excellent | Excellent |
| Learning curve | Medium | Low |

## 🏗 Production Best Practice (REAL WORLD)

**For automation / scripts:**
```bash
✅ oc get -o json | jq …
```

**For human-maintained backups:**
```bash
✅ oc get -o yaml | yq …
```

**For enterprise:**
```bash
❌ Do NOT rely on manual YAML cleanup
✅ Use GitOps / Helm / OADP
```

## ⚠ Things You Should NEVER Remove

Do NOT delete:

- `metadata.name`
- `metadata.namespace`
- `metadata.labels`
- `spec.selector`
- `spec.template`
- PVC `spec.resources`
- Volume definitions

Removing these breaks restore.

## 🎯 Final Architect Rule

**jq** = surgeon
**yq** = editor

Both are valid.
A real OpenShift engineer knows when to use which.