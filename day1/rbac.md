# Core RBAC Objects in OpenShift / Kubernetes

RBAC has these objects:

- Role
- ClusterRole
- RoleBinding
- ClusterRoleBinding
- User
- Group
- ServiceAccount

OpenShift uses Kubernetes RBAC underneath.

## Quick Mental Model

| Scope | Role Type | Binding Type |
|-------|-----------|--------------|
| Namespace | Role | RoleBinding |
| Cluster-wide | ClusterRole | ClusterRoleBinding |

If scope = single project → Role
If scope = entire cluster → ClusterRole

## Users, Groups, ServiceAccounts

### List Users
```bash
oc get users
```

### Describe User
```bash
oc describe user ashu
```

### List Groups
```bash
oc get groups
```

### Describe Group
```bash
oc describe group devops
```

### Add User to Group
```bash
oc adm groups add-users devops ashu
```

### Remove User from Group
```bash
oc adm groups remove-users devops ashu
```

### Service Accounts (Namespace scoped)

List SAs:
```bash
oc get sa -n myproject
```

Create SA:
```bash
oc create sa app-sa -n myproject
```

Describe SA:
```bash
oc describe sa app-sa -n myproject
```

## Roles (Namespace Scoped)

### List Roles
```bash
oc get roles -n myproject
```

### Describe Role
```bash
oc describe role edit -n myproject
```

### Create Custom Role
```bash
oc create role pod-reader \
    --verb=get,list,watch \
    --resource=pods \
    -n myproject
```

## ClusterRoles (Cluster Scoped)

### List ClusterRoles
```bash
oc get clusterroles
```

### Describe ClusterRole
```bash
oc describe clusterrole cluster-admin
```

### Create Custom ClusterRole
```bash
oc create clusterrole node-reader \
    --verb=get,list \
    --resource=nodes
```

## RoleBindings (Namespace Scoped)

Bind role to user:
```bash
oc create rolebinding pod-reader-binding \
    --role=pod-reader \
    --user=ashu \
    -n myproject
```

Bind role to group:
```bash
oc create rolebinding devops-edit \
    --clusterrole=edit \
    --group=devops \
    -n myproject
```

Bind role to ServiceAccount:
```bash
oc create rolebinding sa-binding \
    --clusterrole=edit \
    --serviceaccount=myproject:app-sa \
    -n myproject
```

## ClusterRoleBindings (Cluster Scoped)

Bind cluster-admin to user:
```bash
oc create clusterrolebinding ashu-admin \
    --clusterrole=cluster-admin \
    --user=ashu
```

Bind clusterrole to group:
```bash
oc create clusterrolebinding devops-admin \
    --clusterrole=cluster-admin \
    --group=devops
```

Bind to ServiceAccount:
```bash
oc create clusterrolebinding sa-cluster \
    --clusterrole=cluster-admin \
    --serviceaccount=myproject:app-sa
```

## oc adm policy Commands

These are OpenShift shortcuts.

### Add Role to User (Namespace)
```bash
oc adm policy add-role-to-user edit ashu -n myproject
```

### Add ClusterRole to User (Namespace)
```bash
oc adm policy add-role-to-user admin ashu -n myproject
```

### Add ClusterRole to Group (Namespace)
```bash
oc adm policy add-role-to-group edit devops -n myproject
```

### Add ClusterRole to User (Cluster-wide)
```bash
oc adm policy add-cluster-role-to-user cluster-admin ashu
```

### Add ClusterRole to Group (Cluster-wide)
```bash
oc adm policy add-cluster-role-to-group cluster-admin devops
```

### Add ClusterRole to ServiceAccount

Namespace scope:
```bash
oc adm policy add-role-to-user edit \
    system:serviceaccount:myproject:app-sa \
    -n myproject
```

Cluster scope:
```bash
oc adm policy add-cluster-role-to-user cluster-admin \
    system:serviceaccount:myproject:app-sa
```

## Check Who Has Access

Check if user can perform action:
```bash
oc auth can-i create pods --as=ashu -n myproject
```

Check cluster-wide:
```bash
oc auth can-i get nodes --as=ashu
```

Check using group:
```bash
oc auth can-i create pods --as-group=devops -n myproject
```

## Built-in OpenShift ClusterRoles

Common ones:
- cluster-admin
- admin
- edit
- view
- self-provisioner
- basic-user

List:
```bash
oc get clusterroles
```

## Remove Bindings

Delete rolebinding:
```bash
oc delete rolebinding pod-reader-binding -n myproject
```

Delete clusterrolebinding:
```bash
oc delete clusterrolebinding devops-admin
```

## Advanced — View Effective Permissions

Check full access review:
```bash
oc auth can-i --list --as=ashu -n myproject
```

## Export RBAC YAML
```bash
oc get role edit -n myproject -o yaml
oc get rolebinding devops-edit -n myproject -o yaml
```

## Difference Summary

- **Role** → Namespace only
- **ClusterRole** → Entire cluster
- **RoleBinding** → Binds Role in namespace
- **ClusterRoleBinding** → Binds ClusterRole cluster-wide

ServiceAccount always namespaced.

## Architect-Level Tip

Never give cluster-admin directly to users.

Best practice:
```
LDAP/OIDC group → ClusterRoleBinding → controlled access
```