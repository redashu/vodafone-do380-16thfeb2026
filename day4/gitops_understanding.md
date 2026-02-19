# GitOps Principles

## What is GitOps?

GitOps is an operational model where:

- Git becomes the single source of truth for cluster state.
- Instead of manually running `oc apply`, `kubectl edit`, `oc patch`
- You declare desired state in Git.
- Cluster automatically matches that state.

## Core Idea

**Traditional approach:**
```
Admin → CLI → Cluster
```

**GitOps approach:**
```
Git Repo → GitOps Controller → Cluster
```

Humans interact with Git, NOT directly with the cluster.

## Fundamental Principle

- Desired state lives in Git.
- Actual state lives in cluster.
- GitOps reconciles both.

If cluster drifts → GitOps fixes it automatically.

## Why GitOps Exists

**Manual operations cause:**
- Configuration drift
- Unknown changes
- Hard rollbacks
- Audit issues
- Human errors

**GitOps provides:**
- Declarative infrastructure
- Version control
- Rollback capability
- Auditing
- Automation

## Four Core GitOps Principles

### 1. Declarative Configuration

Everything described as YAML:
- Apps
- Services
- Routes
- RBAC
- Policies

No imperative commands needed.

### 2. Version Controlled

Git records:
- Who changed
- What changed
- When changed

This gives compliance and auditability.

### 3. Automated Synchronization

Controller continuously checks: `Git state == Cluster state?`

If not → reconcile.

### 4. Continuous Reconciliation

Even if someone changes cluster manually, GitOps will revert it back. Git always wins.

## Key Mindset Shift

**Old mindset:** Cluster is source of truth.

**GitOps mindset:** Git is source of truth. Cluster is just runtime.

## Example Flow

1. Developer updates YAML: `replicas: 2 → replicas: 5`
2. Commit → Push → Git
3. GitOps controller detects change
4. Cluster automatically updates deployment
5. No manual action

## Typical GitOps Architecture

```
Git Repository
    ↓
ArgoCD (GitOps Engine)
    ↓
OpenShift Cluster
    ↓
Applications
```

## Drift Detection

If someone runs `oc scale deploy app --replicas=10` but Git says `replicas=3`, GitOps automatically restores `replicas=3`. This prevents configuration drift.

## Benefits in Enterprise

- Repeatable environments
- Easy disaster recovery
- Multi-cluster consistency
- Security & compliance
- Clear audit trail


## Common Misconception

GitOps ≠ CI/CD pipeline.
- CI/CD builds artifacts.
- GitOps deploys desired state continuously.

## Summary

GitOps means:
- Git defines desired state
- Cluster reconciled automatically
- Continuous synchronization
- Automated drift correction
- Version-controlled infrastructure