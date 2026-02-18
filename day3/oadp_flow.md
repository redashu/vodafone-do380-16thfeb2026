# ✅ OVERALL FLOW — OADP Setup to Backup

**Running on:**  
👉 Red Hat OpenShift

**Backup engine based on:**  
👉 Velero

---

## 🔵 STEP 1 — Install OADP Operator

From OperatorHub:

- Install **OADP Operator**
- **Namespace**: `openshift-adp`
- **Approval**: Automatic (recommended)

### What this does:

- Installs Velero CRDs
- Deploys OADP controller
- Prepares cluster for backup
- But does **NOT** configure storage yet

### Verify:

```bash
oc get pods -n openshift-adp
```

---

## 🔵 STEP 2 — Prepare Object Storage

You need a bucket to store backups.

Two scenarios:

### 🟢 Option A — Using ODF (Recommended in OpenShift)

If using:  
👉 Red Hat OpenShift Data Foundation

Create **ObjectBucketClaim**:

```yaml
apiVersion: objectbucket.io/v1alpha1
kind: ObjectBucketClaim
metadata:
  name: backup
  namespace: openshift-adp
spec:
  storageClassName: openshift-storage.noobaa.io
  generateBucketName: backup
```

Apply:

```bash
oc apply -f obc.yaml
```

This automatically:

- Creates S3 bucket
- Creates Secret
- Creates ConfigMap

### 🟢 Option B — External S3 (AWS / MinIO)

If using:  
👉 Amazon S3

Manually:

- Create bucket
- Create credentials secret in `openshift-adp`

---

## 🔵 STEP 3 — Label VolumeSnapshotClass (If Using CSI)

Check snapshot classes:

```bash
oc get volumesnapshotclass
```

Label the one to be used by Velero:

```bash
oc label volumesnapshotclass <snapshot-class-name> \
  velero.io/csi-volumesnapshot-class=true --overwrite
```

This tells Velero which snapshot class to use.

---

## 🔵 STEP 4 — Create DataProtectionApplication (DPA)

This is the **most important step**.

### Example:

```yaml
apiVersion: oadp.openshift.io/v1alpha1
kind: DataProtectionApplication
metadata:
  name: dpa
  namespace: openshift-adp
spec:
  backupLocations:
    - velero:
        provider: aws
        default: true
        objectStorage:
          bucket: <bucket-name>
        credential:
          name: <generated-secret>
          key: cloud
  configuration:
    velero:
      defaultPlugins:
        - aws
        - openshift
        - csi
```

Apply:

```bash
oc apply -f dpa.yaml
```

### What DPA Does:

Operator:

- Configures Velero
- Creates BackupStorageLocation
- Creates VolumeSnapshotLocation
- Connects to bucket
- Enables CSI

### Verify:

```bash
oc get backupstoragelocation -n openshift-adp
```

Must show:

```
AVAILABLE: true
```

---

## 🔵 STEP 5 — Create Backup

Now everything is ready.

Backup namespace `my-app`:

```bash
oc create backup my-app-backup \
  --include-namespaces=my-app \
  --ttl=720h \
  -n openshift-adp
```

---

## 🔵 STEP 6 — Monitor Backup

```bash
oc get backup -n openshift-adp
```

Describe:

```bash
oc describe backup my-app-backup -n openshift-adp
```

Logs:

```bash
oc logs deployment/velero -n openshift-adp
```

---

## 🧠 WHAT HAPPENS INTERNALLY

1️⃣ Velero reads namespace resources  
2️⃣ Exports YAML to object storage  
3️⃣ If PVC exists → creates CSI snapshot  
4️⃣ Snapshot stored at storage backend  
5️⃣ Metadata stored in bucket  

**Backup complete ✅**

---

## 🎯 FINAL SUMMARY FLOW

```
Install OADP Operator
        ↓
Prepare Object Storage (OBC or S3)
        ↓
Label VolumeSnapshotClass
        ↓
Create DataProtectionApplication
        ↓
Verify BackupStorageLocation
        ↓
Create Backup
```

---

## 🏁 One-Line Executive Summary

**Install operator → configure storage → create DPA → verify BSL → create Backup resource**