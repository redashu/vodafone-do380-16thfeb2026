# Why Backup in OpenShift is Different from Traditional Systems

OpenShift is:

- Declarative
- Distributed
- API-driven
- Stateful + stateless mix
- Etcd-backed control plane

You must protect:

- Kubernetes objects (API resources)
- Persistent data (PVC / storage)
- Application state
- Metadata & relationships
- Cluster configuration

That's why traditional VM backup tools are not enough.

## What is OADP?

**OADP** = OpenShift API for Data Protection

It is Red Hat's supported framework for:

- Backup
- Restore
- Migration
- Disaster Recovery

It is based on Velero.

**OADP = Velero + OpenShift integration + supported operators.**

## Core Problem OADP Solves

You want to:

- Backup a namespace
- Restore a deleted app
- Migrate workloads to another cluster
- Recover from cluster failure
- Clone environment (dev → stage)
- Perform DR testing

OADP enables this in a Kubernetes-native way.

## Logical Layers in OADP Architecture

There are two types of data:

### A) Kubernetes Objects

Examples:

- Deployment
- StatefulSet
- Service
- ConfigMap
- Secrets
- Routes
- RBAC
- CRDs

These are stored in etcd. OADP backs them up via API.

### B) Persistent Volume Data

Data inside:

- PVC
- Stateful app storage
- Databases

This requires storage snapshot or file copy mechanism.

## Conceptual Architecture

OADP consists of:

- Operator
- Velero server
- Backup storage location (S3-compatible)
- Volume snapshot location
- Restic or CSI snapshot integration

**Flow:**
```
Cluster → OADP → Object Storage → Backup artifacts
```

## What Gets Backed Up?

Backup includes:

- Namespaces
- Custom Resources
- CRDs (optional)
- Persistent Volume Claims
- Persistent Volume snapshots
- Application metadata

**Important:**

- Node-level OS is NOT backed up.
- Control plane nodes are NOT backed up via OADP.
- OADP is workload-level backup.

## Types of Backup

- **Namespace-level Backup** - Most common.
- **Label-based Backup** - Select resources using label selector.
- **Cluster-scoped Backup** - Includes cluster roles, CRDs, etc.
- **Scheduled Backup** - Automated periodic backups.

## Restore Concepts

Restore is NOT just "apply YAML."

Restore includes:

- Recreating objects
- Rebinding PVC
- Restoring snapshots
- Re-creating RBAC
- Handling name remapping
- Conflict resolution

Restore can be:

- Same namespace
- Different namespace
- Different cluster

## Migration Concept

**Migration** = Backup from cluster A → Restore in cluster B.

This is used for:

- Cluster upgrade
- Data center migration
- Cloud migration
- Region failover

It requires:

- Compatible storage
- Same CRDs installed
- Matching operator versions

## Backup Storage Location

Backups are stored in:

- S3
- S3-compatible (MinIO)
- AWS
- Azure Blob
- GCP

Backups stored as:

- Metadata files
- JSON/YAML resource definitions
- Snapshot references

Cluster must access object storage.

## Volume Backup Methods

Two approaches:

### A) CSI Snapshot-based

- Fast, storage-level snapshot
- Best for cloud-native storage

### B) Restic-based

- File-level backup
- Works when snapshots not supported
- Slower

## Disaster Recovery Models

There are 3 DR models:

1. **Level 1 – Application-level Restore** - Restore single app
2. **Level 2 – Namespace Restore** - Restore full namespace
3. **Level 3 – Cross-Cluster Migration** - Restore into new cluster

Full cluster etcd disaster recovery is separate from OADP.

## What OADP Does NOT Replace

It does NOT replace:

- etcd backup (control plane backup)
- Node OS backup
- Infrastructure backup (load balancer, DNS)
- Machine API recovery

OADP is workload-focused.

## Critical Design Considerations

Before implementing OADP, architect must decide:

- RPO (Recovery Point Objective)
- RTO (Recovery Time Objective)
- Storage backend
- Snapshot capability
- Cross-region strategy
- Encryption at rest
- Backup retention policy

## Common Enterprise Use Cases

- Backup before OpenShift upgrade
- Protect production namespace daily
- Clone production into staging
- Recover deleted namespace
- Migrate from on-prem to cloud
- DR compliance testing

## Security Considerations

Backups contain:

- Secrets
- ConfigMaps
- ServiceAccount tokens
- Possibly database dumps

Therefore:

- Encrypt backup storage
- Restrict S3 access
- Use IAM roles
- Protect restore permissions

## What Happens During Backup (Internal Concept)

- Velero queries API server
- Collects resource definitions
- Stores metadata
- Triggers snapshot or Restic
- Uploads to object storage
- Records Backup CR status

Everything is Kubernetes-native.

## Restore Flow Internally

- User triggers Restore CR
- Velero reads backup metadata
- Recreates resources
- Restores PVC
- Handles conflicts
- Updates status

## Architectural Separation

**Control Plane DR:**
- etcd snapshot
- Separate process

**Application DR:**
- OADP
- Never confuse both.

## Migration vs Backup Difference

**Backup:**
- Protect same cluster.

**Migration:**
- Move workloads across clusters.

Migration includes:

- Image registry access
- Storage compatibility
- CRD availability
