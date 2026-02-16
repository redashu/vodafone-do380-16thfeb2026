# 🧠 What is LDAP?

LDAP = Lightweight Directory Access Protocol

It is a protocol used to:

- Store identities (users, groups)
- Authenticate users
- Provide centralized directory services

Think of LDAP as:

- A hierarchical database optimized for identity lookup.
- It is NOT just authentication — it is a directory.

## 🏢 Where LDAP is Used

Most enterprises use:

- Microsoft Active Directory (which supports LDAP)
- Red Hat Directory Server
- OpenLDAP
- 389 Directory Server

LDAP is everywhere in corporate networks.

## 🏗 LDAP Architecture Basics

LDAP has:

```
Client (OpenShift)
↓
LDAP Server
↓
Directory Database
```

OpenShift acts as LDAP client.

## 📂 LDAP Data Model (VERY IMPORTANT)

LDAP stores data in a tree structure called:

**DIT – Directory Information Tree**

Example structure:

```
dc=corp,dc=com
│
├── ou=users
│     ├── uid=ashu
│     ├── uid=dev1
│
├── ou=groups
    ├── cn=devops
    ├── cn=admins
```

This is hierarchical.

## 🔑 Key LDAP Concepts You Must Understand

### 1️⃣ DN – Distinguished Name

The full path of an object.

Example:

```
uid=ashu,ou=users,dc=corp,dc=com
```

This uniquely identifies a user.

### 2️⃣ RDN – Relative Distinguished Name

The first part:

```
uid=ashu
```

### 3️⃣ DC – Domain Component

```
dc=corp
dc=com
```

Represents domain hierarchy.

### 4️⃣ OU – Organizational Unit

Logical grouping:

```
ou=users
ou=groups
```

### 5️⃣ Attributes

Each entry has attributes:

Example user entry:

```
uid: ashu
cn: Ashu Sharma
mail: ashu@corp.com
objectClass: inetOrgPerson
```

Attributes are key-value pairs.

### 6️⃣ objectClass

Defines schema type of entry.

Common ones:

- inetOrgPerson
- organizationalUnit
- groupOfNames
- posixAccount

This controls allowed attributes.

## 🔐 How LDAP Authentication Works

OpenShift does:

1. Bind (login) to LDAP using bindDN
2. Search for user using filter
3. Try to authenticate user by binding as that user

Flow:

```
OpenShift → Bind as service account
Search for uid=ashu
Get DN
Bind as uid=ashu using user password
If bind successful → authenticated
```

LDAP does NOT issue tokens.
It just validates credentials.

## 🔎 LDAP Search Components (Very Important for Config)

When configuring LDAP in OpenShift, you must know:

### 1️⃣ URL

Example:

```
ldaps://ldap.corp.com:636/ou=users,dc=corp,dc=com?uid
```

Breakdown:

- Protocol: ldaps
- Base DN: ou=users,dc=corp,dc=com
- Attribute to match: uid

### 2️⃣ Search Filter

Example:

```
(uid={0})
```

`{0}` is replaced with username.

### 3️⃣ Bind DN

Service account used to search directory:

```
cn=ldap-reader,dc=corp,dc=com
```

### 4️⃣ Bind Password

Stored as secret in OpenShift.

## 🔐 LDAP Security

Two protocols:

- LDAP (389) → Not encrypted
- LDAPS (636) → Encrypted

Production rule:
Always use LDAPS or StartTLS.

OpenShift must trust LDAP server certificate.

## 👥 LDAP Groups

Groups stored like:

```
cn=devops,ou=groups,dc=corp,dc=com
```

Attributes:

```
member: uid=ashu,ou=users,dc=corp,dc=com
member: uid=dev1,ou=users,dc=corp,dc=com
```

OpenShift can sync these groups.

## 🧪 Commands to Explore LDAP (If You Had Access)

```bash
ldapsearch -x -H ldaps://ldap.corp.com \
-D "cn=ldap-reader,dc=corp,dc=com" \
-W \
-b "ou=users,dc=corp,dc=com"
```

This is critical for troubleshooting.

## 🏗 Minimal Knowledge Required Before Configuring LDAP in OCP

You must know:

- LDAP server FQDN
- Port
- Base DN
- Bind DN
- Bind password
- User search attribute (uid? sAMAccountName?)
- Group base DN
- Group membership attribute (member? memberOf?)
- Certificate trust chain