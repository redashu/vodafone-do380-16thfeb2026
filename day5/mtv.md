# Migration Toolkit for Virtualization (MTV)

## What is MTV?

Migration Toolkit for Virtualization (MTV) is an OpenShift tool used to migrate virtual machines from external virtualization platforms into OpenShift Virtualization. Think of it as a VM Migration Automation Platform.

## Problem MTV Solves

Without MTV, migrating a VM requires:
- Manual disk copy
- Network reconfiguration
- VM recreation
- Driver adjustments
- High risk and downtime

MTV automates this entire workflow.

## High-Level Architecture

```
Source Platform (VMware / others)
            ↓
        MTV Operator
            ↓
OpenShift Virtualization (Target)
```

MTV handles:
- VM discovery
- Disk migration
- Network mapping
- VM conversion

## Supported Source Platforms

Common sources:
- VMware vSphere (MOST COMMON)
- Red Hat Virtualization (RHV)
- OpenStack (depending on version)

DO380 usually focuses on VMware → OpenShift.

## Core MTV Components

### Provider
Represents source or destination (e.g., VMware provider, OpenShift provider). Defines connection details.

### Inventory
MTV discovers datacenters, clusters, VMs, networks, and storage from source platform.

### Plan
Migration plan defines which VMs to migrate, how networking maps, and how storage maps. This is the blueprint.

### Migration
Actual execution of migration plan. Copies data and creates VM in OpenShift.

## Migration Workflow

```
Add Source Provider → Discover VMs → Create Migration Plan
        ↓
Map Networks & Storage → Execute Migration → VM appears in OpenShift
```

## Types of Migration

**Cold Migration**: VM powered OFF during migration. Most reliable, low risk, common in training.

**Warm Migration**: Incremental copy while VM runs. Minimal downtime, more advanced.

## Why MTV is Kubernetes-Native

After migration, VMs become `VirtualMachine` CRs and are managed like any other OpenShift resource.

## What MTV Automatically Handles

- Disk transfer
- VM metadata conversion
- Network interface mapping
- Storage conversion
- Creation of DataVolumes

## Storage & Network Handling

VM disks become PersistentVolumeClaims inside OpenShift. VMware networks must map to OpenShift or Multus networks via Network Mapping.

## What MTV Does NOT Do

- OS driver optimization
- Application reconfiguration
- Performance tuning
- Legacy hardware compatibility

Post-migration validation is always required.

## Summary

MTV provides:
- ✔ Automated VM discovery
- ✔ Migration planning
- ✔ Storage/network mapping
- ✔ VM conversion into OpenShift
- ✔ Cold & warm migration support
