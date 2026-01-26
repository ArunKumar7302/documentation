# Kubernetes RBAC (Role Based Access Control) – Practical Guide

This document explains Kubernetes RBAC using a simple real-world scenario.  
RBAC allows us to control **who can do what** inside a Kubernetes cluster.

---

## Why RBAC is Needed

Assume you have a Kubernetes cluster and three people in your company:

| Person | Responsibility |
|------|--------------|
| Tester | Only check if Pods are running |
| Developer | Check Pod logs |
| DevOps Engineer | Full control over cluster |

By default, if all of them have kubeconfig access, they can:
- Delete pods
- Modify deployments
- Break production

This is dangerous.

We want:
- Tester → Only view pod status
- Developer → Only read pod logs
- DevOps → Full permissions

That is exactly what RBAC does.

RBAC = **Role Based Access Control**  
Access is decided based on Roles.

---

## Another Problem RBAC Solves

Suppose we have a Pod whose job is:

> Check if all pods in the cluster are running.  
> If any pod is down, send an email.

By default:
- A pod cannot see other pods
- A pod has zero permissions
- This is for security

So we must give that pod limited access:
- Only read pod status
- No delete
- No modify

This is also done using RBAC.

---

## Core RBAC Components

| Component | Purpose |
|--------|-------|
| Role | Defines permissions inside a namespace |
| RoleBinding | Assigns Role to a user or service account |
| ClusterRole | Defines permissions cluster-wide |
| ClusterRoleBinding | Assigns ClusterRole |
| ServiceAccount | Identity used by Pods |

---

## 1. Role

Role defines what actions are allowed.

Example:
Allow reading Pod status:
```bash
apiVersion: rbac.authorization.k8s.io/v1  
kind: Role  
metadata:  
  name: pod-kavalan-role  
  namespace: default  
rules:  
  - apiGroups: [""]  
    resources: ["pods"]  
    verbs: ["get", "list", "watch"]  
```
Meaning:

| Verb | Purpose |
|----|-------|
| get | Read a single pod |
| list | Get list of pods |
| watch | Monitor live status changes |

This role allows only **read operations** on pods.

---

## 2. Service Account

Pods cannot be bound directly to Roles.  
They use a ServiceAccount.

ServiceAccount creation:
```bash
apiVersion: v1  
kind: ServiceAccount  
metadata:  
  name: pod-kavalan-sa  
  namespace: default  
```
Think of ServiceAccount as:
> A user identity for Pods

---

## 3. RoleBinding

RoleBinding connects:
Role → ServiceAccount
```bash
apiVersion: rbac.authorization.k8s.io/v1  
kind: RoleBinding  
metadata:  
  name: pod-kavalan-role-binding  
  namespace: default  
subjects:  
  - kind: ServiceAccount  
    name: pod-kavalan-sa  
    namespace: default  
roleRef:  
  kind: Role  
  name: pod-kavalan-role  
  apiGroup: rbac.authorization.k8s.io  
```
Now:
ServiceAccount `pod-kavalan-sa` has the permissions of `pod-kavalan-role`.

---

## 4. Assign ServiceAccount to Pod

In Deployment:
```bash
apiVersion: apps/v1  
kind: Deployment  
metadata:  
  name: pod-kavalan-deployment  
  namespace: default  
spec:  
  replicas: 1  
  selector:  
    matchLabels:  
      app: pod-kavalan  
  template:  
    metadata:  
      labels:  
        app: pod-kavalan  
    spec:  
      serviceAccountName: pod-kavalan-sa  
      containers:  
      - name: app  
        image: nginx  
```
Now this pod:
- Can see other pods
- Can check their status
- Cannot delete or modify anything

Perfect security.

---

## Mapping to Human Users

| Role | Permissions |
|-----|-----------|
| Tester | get, list pods |
| Developer | get pods + get logs |
| DevOps | Full access (*) |

Each user gets:
- A Role
- A RoleBinding

---

## Role vs ClusterRole

| Role | ClusterRole |
|----|-----------|
| Namespace scoped | Cluster scoped |
| For pods, secrets, configmaps | For nodes, PVs, namespaces |
| Requires namespace | No namespace |

Resources that are cluster-wide:
- Nodes
- PersistentVolumes
- Namespaces

For them, we use:
`kind: ClusterRole`  
`kind: ClusterRoleBinding  
`
Everything else works the same.

---

## Final Summary

RBAC helps us to:

| Problem | Solution |
|------|-------|
| Prevent accidental deletes | Restrict permissions |
| Give minimal access | Use Roles |
| Secure pods | Use ServiceAccounts |
| Control cluster security | Use ClusterRoles |

RBAC ensures:
> Right person / pod gets the right permission and nothing more.
