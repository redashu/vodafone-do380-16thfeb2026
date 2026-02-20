# Red Hat Advanced Cluster Management (ACM) for Kubernetes

Complete guide to managing multiple OpenShift clusters from a central hub.

---

## 🔷 What is ACM (Simple Meaning)

**ACM** =

> One central hub cluster to manage many OpenShift clusters.

Think like:

```
Hub Cluster (ACM installed here)
        |
        |---- Managed Cluster 1 (Dev)
        |---- Managed Cluster 2 (Stage)
        |---- Managed Cluster 3 (Prod)
```

**Instead of logging into each cluster, you control all from the HUB.**

---

## 1️⃣ Multi-Cluster Lifecycle Management

### What it means

Managing:

- Cluster creation
- Cluster import
- Cluster upgrades
- Cluster deletion

**From a single hub.**

### Example Scenario

Your company has:

```
5 clusters (dev, stage, prod, dr, qa)
```

**Without ACM:**

- You manage each separately

**With ACM:**

- You upgrade all clusters from one console

### 🔹 Demo – Import an Existing Cluster

On hub cluster:

```bash
oc get managedclusters
```

To import:

1. Go to **ACM console → Infrastructure → Clusters**
2. Click "**Import Cluster**"
3. Download import YAML
4. Apply on target cluster:

```bash
oc apply -f import.yaml
```

Now:

```bash
oc get managedclusters
```

You will see cluster as:

```
Ready=True
```

**That's lifecycle management.**

---

## 2️⃣ Governance, Risk & Compliance (GRC)

> ⚠️ This is **VERY IMPORTANT** for DO380.

ACM allows you to:

**Enforce policies across all clusters.**

### Example:

- ✅ All clusters must disable privileged pods
- ✅ All clusters must use specific NTP server
- ✅ All clusters must not allow default namespace deployments

### How It Works

ACM uses:

- **Policy**
- **Placement**
- **PlacementBinding**

### 🔹 Demo – Enforce No Privileged Pods

Create a Policy on Hub:

```yaml
apiVersion: policy.open-cluster-management.io/v1
kind: Policy
metadata:
  name: disallow-privileged
  namespace: default
spec:
  remediationAction: enforce
  policy-templates:
    - objectDefinition:
        apiVersion: constraints.gatekeeper.sh/v1beta1
        kind: K8sPSPPrivilegedContainer
        metadata:
          name: psp-privileged-container
```

Then bind it to clusters.

**Now:**

If any cluster creates privileged pod, ACM flags it **NonCompliant**.

---

## 3️⃣ Integrating ArgoCD with ACM

ACM supports **GitOps model**.

Uses:  
👉 **Argo CD**

### Concept:

```
Git Repo → ACM → Deploy to Multiple Clusters
```

Instead of manually applying YAML everywhere.

### 🔹 Demo – GitOps Deployment

1. Install **OpenShift GitOps operator**
2. Create **GitOpsCluster CR**
3. Point to Git repo

Example ApplicationSet:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: multi-cluster-app
  namespace: openshift-gitops
spec:
  generators:
  - clusterDecisionResource:
      configMapRef: acm-placement
      labelSelector:
        matchLabels:
          cluster.open-cluster-management.io/placement: app-placement
      requeueAfterSeconds: 180
  template:
    metadata:
      name: '{{name}}-app'
    spec:
      project: default
      source:
        repoURL: https://github.com/myorg/myapp
        targetRevision: HEAD
        path: k8s
      destination:
        server: '{{server}}'
        namespace: myapp
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

**Now:**

Same app deploys to dev, stage, prod automatically.

**That's multi-cluster GitOps.**

---

## 4️⃣ Cluster Import & Management Workflows

This means:

- Add cluster
- Label cluster
- Group cluster
- Apply policies
- Upgrade cluster
- Delete cluster

### 🔹 Example Workflow

#### Step 1: Import cluster

```bash
# Download kubeconfig for target cluster
# Apply import command from ACM console
```

#### Step 2: Label it

```bash
oc label managedcluster dev-cluster environment=dev
```

#### Step 3: Create Placement

```yaml
apiVersion: cluster.open-cluster-management.io/v1beta1
kind: Placement
metadata:
  name: dev-placement
  namespace: default
spec:
  clusterSets:
    - default
  predicates:
  - requiredClusterSelector:
      labelSelector:
        matchLabels:
          environment: dev
```

**Now policies apply only to dev clusters.**

---

## 🔥 Real Enterprise Example

### Company structure:

```
Hub Cluster (ACM)
   |
   ├── dev-us
   ├── dev-eu
   ├── prod-us
   ├── prod-eu
```

### Using ACM:

- ✅ Apply security policy to all prod clusters
- ✅ Deploy app to only dev clusters
- ✅ Upgrade only eu clusters
- ✅ Detect drift across clusters

---

## 🏗️ ACM Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Hub Cluster                           │
│  (ACM MultiClusterHub Installed)                        │
├─────────────────────────────────────────────────────────┤
│                                                           │
│  ┌───────────────────────────────────────────────────┐  │
│  │           ACM Components                          │  │
│  │                                                    │  │
│  │  • MultiClusterHub Operator                       │  │
│  │  • Policy Controllers                             │  │
│  │  • Application Lifecycle                          │  │
│  │  • Observability                                  │  │
│  │  • Search                                         │  │
│  │  • Governance & Risk                              │  │
│  └───────────────────────────────────────────────────┘  │
│                          ↓                               │
│  ┌───────────────────────────────────────────────────┐  │
│  │         Managed Cluster Registration              │  │
│  │  (Klusterlet agents on managed clusters)          │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
                          ↓
        ┌─────────────────┼─────────────────┐
        ↓                 ↓                 ↓
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ Managed      │  │ Managed      │  │ Managed      │
│ Cluster 1    │  │ Cluster 2    │  │ Cluster 3    │
│ (Dev)        │  │ (Stage)      │  │ (Prod)       │
│              │  │              │  │              │
│ Klusterlet   │  │ Klusterlet   │  │ Klusterlet   │
│ Agent        │  │ Agent        │  │ Agent        │
└──────────────┘  └──────────────┘  └──────────────┘
```

---

## 📦 ACM Components

### Hub Cluster Components

| Component | Purpose |
|-----------|---------|
| **MultiClusterHub** | Main ACM operator |
| **Policy Controller** | Enforces governance policies |
| **Application Manager** | Manages multi-cluster applications |
| **Cluster Lifecycle** | Manages cluster provisioning and import |
| **Observability** | Collects metrics from managed clusters |
| **Search** | Indexes resources across clusters |
| **Console** | Web UI for management |

### Managed Cluster Components

| Component | Purpose |
|-----------|---------|
| **Klusterlet** | Agent that registers cluster with hub |
| **Policy Agent** | Enforces policies locally |
| **Application Agent** | Deploys applications |
| **Metrics Collector** | Sends metrics to hub |

---

## 🛠️ Installation

### Step 1: Install ACM Operator

On the **hub cluster**:

1. Navigate to **OperatorHub**
2. Search for "**Advanced Cluster Management for Kubernetes**"
3. Install in namespace: `open-cluster-management`
4. Wait for operator to be ready

```bash
oc get csv -n open-cluster-management
```

### Step 2: Create MultiClusterHub

```yaml
apiVersion: operator.open-cluster-management.io/v1
kind: MultiClusterHub
metadata:
  name: multiclusterhub
  namespace: open-cluster-management
spec: {}
```

Apply:

```bash
oc apply -f multiclusterhub.yaml
```

### Step 3: Verify Installation

```bash
# Check MultiClusterHub status
oc get multiclusterhub -n open-cluster-management

# Check all ACM components
oc get pods -n open-cluster-management

# Access ACM Console
oc get route multicloud-console -n open-cluster-management
```

---

## 🔍 Working with Managed Clusters

### Import an Existing Cluster

#### Method 1: Via Console

1. Go to **Infrastructure → Clusters**
2. Click **Import cluster**
3. Enter cluster name
4. Download generated command
5. Run on target cluster

#### Method 2: Via CLI

**On Hub:**

Create ManagedCluster:

```yaml
apiVersion: cluster.open-cluster-management.io/v1
kind: ManagedCluster
metadata:
  name: dev-cluster
  labels:
    environment: dev
    region: us-east
spec:
  hubAcceptsClient: true
```

**On Managed Cluster:**

Get import command:

```bash
oc get secret dev-cluster-import -n dev-cluster -o jsonpath='{.data.import\.yaml}' | base64 -d | oc apply -f -
```

### Create a New Cluster

ACM can provision new clusters on:

- AWS
- Azure
- GCP
- VMware
- Bare Metal
- Red Hat OpenStack

**Example: Create AWS Cluster**

```yaml
apiVersion: hive.openshift.io/v1
kind: ClusterDeployment
metadata:
  name: prod-cluster
  namespace: prod-cluster
spec:
  baseDomain: example.com
  clusterName: prod-cluster
  platform:
    aws:
      region: us-east-1
  provisioning:
    installConfigSecretRef:
      name: prod-cluster-install-config
    imageSetRef:
      name: openshift-v4.12.0
```

---

## 🛡️ Governance & Compliance

### Policy Structure

```yaml
apiVersion: policy.open-cluster-management.io/v1
kind: Policy
metadata:
  name: policy-namespace
  namespace: default
spec:
  remediationAction: inform  # or "enforce"
  disabled: false
  policy-templates:
  - objectDefinition:
      apiVersion: policy.open-cluster-management.io/v1
      kind: ConfigurationPolicy
      metadata:
        name: policy-namespace-prod
      spec:
        remediationAction: inform
        severity: low
        object-templates:
        - complianceType: musthave
          objectDefinition:
            apiVersion: v1
            kind: Namespace
            metadata:
              name: prod
```

### Placement for Policies

```yaml
apiVersion: cluster.open-cluster-management.io/v1beta1
kind: Placement
metadata:
  name: prod-placement
  namespace: default
spec:
  predicates:
  - requiredClusterSelector:
      labelSelector:
        matchExpressions:
        - key: environment
          operator: In
          values:
          - prod
```

### PlacementBinding

```yaml
apiVersion: policy.open-cluster-management.io/v1
kind: PlacementBinding
metadata:
  name: binding-policy-namespace
  namespace: default
placementRef:
  name: prod-placement
  kind: Placement
  apiGroup: cluster.open-cluster-management.io
subjects:
- name: policy-namespace
  kind: Policy
  apiGroup: policy.open-cluster-management.io
```

---

## 📊 Common Policy Examples

### 1. Require Namespace Labels

```yaml
apiVersion: policy.open-cluster-management.io/v1
kind: Policy
metadata:
  name: policy-namespace-labels
spec:
  remediationAction: enforce
  policy-templates:
  - objectDefinition:
      apiVersion: policy.open-cluster-management.io/v1
      kind: ConfigurationPolicy
      metadata:
        name: require-namespace-labels
      spec:
        object-templates:
        - complianceType: musthave
          objectDefinition:
            apiVersion: v1
            kind: Namespace
            metadata:
              labels:
                owner: required
```

### 2. Enforce Pod Security Standards

```yaml
apiVersion: policy.open-cluster-management.io/v1
kind: Policy
metadata:
  name: policy-pod-security
spec:
  remediationAction: enforce
  policy-templates:
  - objectDefinition:
      apiVersion: policy.open-cluster-management.io/v1
      kind: ConfigurationPolicy
      metadata:
        name: pod-security-restricted
      spec:
        object-templates:
        - complianceType: musthave
          objectDefinition:
            apiVersion: v1
            kind: Namespace
            metadata:
              labels:
                pod-security.kubernetes.io/enforce: restricted
```

### 3. Require Certificate Manager

```yaml
apiVersion: policy.open-cluster-management.io/v1
kind: Policy
metadata:
  name: policy-cert-manager
spec:
  remediationAction: inform
  policy-templates:
  - objectDefinition:
      apiVersion: policy.open-cluster-management.io/v1
      kind: ConfigurationPolicy
      metadata:
        name: cert-manager-installed
      spec:
        object-templates:
        - complianceType: musthave
          objectDefinition:
            apiVersion: operators.coreos.com/v1alpha1
            kind: Subscription
            metadata:
              name: cert-manager
              namespace: cert-manager
```

---

## 🚀 Application Deployment

### Deploy to Multiple Clusters

```yaml
apiVersion: app.k8s.io/v1beta1
kind: Application
metadata:
  name: nginx-app
  namespace: default
spec:
  componentKinds:
  - group: apps.open-cluster-management.io
    kind: Subscription
  selector:
    matchLabels:
      app: nginx-app
---
apiVersion: apps.open-cluster-management.io/v1
kind: Channel
metadata:
  name: nginx-channel
  namespace: default
spec:
  type: Git
  pathname: https://github.com/myorg/nginx-app.git
---
apiVersion: apps.open-cluster-management.io/v1
kind: Subscription
metadata:
  name: nginx-subscription
  namespace: default
spec:
  channel: default/nginx-channel
  placement:
    placementRef:
      name: dev-placement
```

---

## 🔧 Troubleshooting

### Check Cluster Status

```bash
# List all managed clusters
oc get managedclusters

# Check specific cluster
oc describe managedcluster dev-cluster

# Check klusterlet status
oc get klusterletaddonconfig -n dev-cluster
```

### Check Policy Compliance

```bash
# List all policies
oc get policies -A

# Check policy status
oc describe policy policy-namespace -n default

# View compliance on managed cluster
oc get configurationpolicies -A
```

### Check Application Status

```bash
# List applications
oc get applications -A

# Check subscriptions
oc get subscriptions.apps.open-cluster-management.io -A

# View placement decisions
oc get placementdecisions -A
```

### Common Issues

#### Cluster Not Joining

```bash
# Check import secret
oc get secret -n <cluster-name>

# Check klusterlet logs on managed cluster
oc logs -n open-cluster-management-agent deploy/klusterlet
```

#### Policy Not Enforcing

```bash
# Check placement
oc get placement -A

# Check placement binding
oc get placementbinding -A

# Verify cluster labels
oc get managedcluster <name> --show-labels
```

---

## 🔎 What DO380 May Ask

**Not theory like:**

- ❌ What is ACM?

**Instead:**

- ✅ Import a cluster
- ✅ Create a policy
- ✅ Make it enforce
- ✅ Verify compliance
- ✅ Use placement to target cluster

---

## 🧠 Simple Summary of Each Topic

| Topic | Simple Meaning |
|-------|----------------|
| **Multi-cluster lifecycle** | Manage many clusters from one |
| **Governance** | Enforce policies centrally |
| **ArgoCD integration** | GitOps multi-cluster deployment |
| **Cluster workflows** | Import, label, group, manage |

---

## 🎯 Final Simple Definition

**ACM is:**

> A centralized control plane to manage, govern, and deploy applications across multiple OpenShift clusters.

---

## 📝 Quick Reference Commands

```bash
# Install ACM
oc create namespace open-cluster-management
oc create -f multiclusterhub.yaml

# Import cluster
oc get secret <cluster-name>-import -n <cluster-name> -o jsonpath='{.data.import\.yaml}' | base64 -d

# List managed clusters
oc get managedclusters

# Label cluster
oc label managedcluster <name> environment=prod

# Create policy
oc apply -f policy.yaml

# Check compliance
oc get policy -A

# Deploy application
oc apply -f application.yaml

# Check placement
oc get placement -A
oc get placementdecisions -A
```

---

## 🏁 Key Takeaways

1. **ACM provides centralized management** for multiple OpenShift clusters
2. **Governance through policies** ensures compliance across all clusters
3. **Placement rules** control which clusters receive policies/apps
4. **GitOps integration** enables declarative multi-cluster deployments
5. **Lifecycle management** simplifies cluster provisioning and upgrades
6. **DO380 focuses on practical skills**: import, policy, placement, compliance

---

## 📚 Additional Resources

- [Official ACM Documentation](https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/)
- [ACM Policy Framework](https://github.com/open-cluster-management-io/policy-collection)
- [GitOps with ACM](https://github.com/open-cluster-management-io/multicloud-gitops)