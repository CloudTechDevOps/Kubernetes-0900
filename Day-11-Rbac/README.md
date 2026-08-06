# Kubernetes RBAC - Complete Guide

<div align="center">

<img src="https://raw.githubusercontent.com/kubernetes/kubernetes/master/logo/logo.png" width="150"/>

# 🔐 Kubernetes RBAC (Role Based Access Control)

### Secure Kubernetes Authorization & Permissions

</div>

---

# 📘 Table of Contents

1. What is RBAC
2. Authentication vs Authorization
3. RBAC Components
4. Role
5. ClusterRole
6. RoleBinding
7. ClusterRoleBinding
8. aws-auth ConfigMap
9. RBAC Architecture
10. Real-Time Use Cases
11. Useful Commands
12. Interview Questions
13. Summary

---

# 1. What is RBAC?

RBAC stands for:

```text
Role Based Access Control
```

RBAC is used to:

- Control user permissions
- Restrict access
- Secure Kubernetes cluster
- Define who can perform actions

---

# Example

| User | Permission |
|---|---|
| DevOps Team | Full cluster access |
| Developers | Namespace access |
| QA Team | Read-only access |

---

# 2. Authentication vs Authorization

| Authentication | Authorization |
|---|---|
| Who are you? | What can you do? |
| User verification | Permission checking |
| IAM/User login | RBAC rules |

---

# 3. RBAC Components

| Component | Purpose |
|---|---|
| Role | Namespace permissions |
| ClusterRole | Cluster-wide permissions |
| RoleBinding | Bind Role to User |
| ClusterRoleBinding | Bind ClusterRole globally |
| ServiceAccount | Pod identity |
| aws-auth ConfigMap | IAM to Kubernetes mapping |

---

# 4. Role

## What is Role?

Role provides permissions inside:
- Specific namespace only

---

## Example Use Cases

- Pod read access
- Deployment management
- ConfigMap access

---

## role.yml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role

metadata:
  namespace: dev
  name: pod-reader

rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

---

## Apply Role

```bash
kubectl apply -f role.yml
```

---

# 5. ClusterRole

## What is ClusterRole?

ClusterRole provides:
- Cluster-wide permissions

Works across:
- All namespaces

---

## Example Use Cases

- Cluster admin
- Node management
- Persistent volume access

---

## crb.yml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole

metadata:
  name: cluster-pod-reader

rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

---

## Apply ClusterRole

```bash
kubectl apply -f crb.yml
```

---

# 6. RoleBinding

## What is RoleBinding?

RoleBinding connects:
- User
- Group
- ServiceAccount

to a Role.

---

## role-bind.yml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding

metadata:
  name: read-pods
  namespace: dev

subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io

roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

## Apply RoleBinding

```bash
kubectl apply -f role-bind.yml
```

---

# 7. ClusterRoleBinding

## What is ClusterRoleBinding?

ClusterRoleBinding provides:
- Cluster-wide access

It binds:
- Users
- Groups
- ServiceAccounts

to ClusterRole.

---

## cluster-role-binding.yml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding

metadata:
  name: cluster-admin-binding

subjects:
- kind: User
  name: admin-user
  apiGroup: rbac.authorization.k8s.io

roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

---

## Apply ClusterRoleBinding

```bash
kubectl apply -f cluster-role-binding.yml
```

---

# 8. aws-auth ConfigMap

## What is aws-auth ConfigMap?

In EKS:
- IAM users/roles must map into Kubernetes RBAC

This mapping is stored in:

```text
aws-auth ConfigMap
```

inside:

```text
kube-system namespace
```

---

## auth.yml Example

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: aws-auth
  namespace: kube-system

data:
  mapRoles: |
    - rolearn: arn:aws:iam::123456789012:role/EKSNodeRole
      username: system:node:{{EC2PrivateDNSName}}
      groups:
        - system:bootstrappers
        - system:nodes

  mapUsers: |
    - userarn: arn:aws:iam::123456789012:user/dev-user
      username: dev-user
      groups:
        - system:masters
```

---

## Apply aws-auth ConfigMap

```bash
kubectl apply -f auth.yml
```

---

# 9. RBAC Architecture

<div align="center">

```text
                 IAM User / Kubernetes User
                               │
                               ▼
                        Authentication
                               │
                               ▼
                            API Server
                               │
                               ▼
                           RBAC Check
                               │
                   ┌─────────────────────┴─────────────────────┐
                   ▼                                           ▼
          
                Role                                      ClusterRole
                   │                                           │
                   ▼                                           ▼
          
             RoleBinding                              ClusterRoleBinding
                   │                                           │
                   └───────────────────────────────────────────┘
                                         │
                                         ▼
                                  Access Granted
```

</div>

---

# 10. Real-Time Use Cases

| Team | Access |
|---|---|
| Developers | Namespace limited access |
| DevOps | Full cluster access |
| Monitoring Team | Read-only access |
| CI/CD Pipeline | Deployment permissions |

---

# Example Scenario

## Developer Access

Developer can:

- View pods
- View logs
- Deploy applications

Developer cannot:

- Delete nodes
- Modify cluster roles

---

# 11. Useful Commands

## View Roles

```bash
kubectl get roles
```

---

## View ClusterRoles

```bash
kubectl get clusterroles
```

---

## View RoleBindings

```bash
kubectl get rolebindings
```

---

## View ClusterRoleBindings

```bash
kubectl get clusterrolebindings
```

---

## Describe Role

```bash
kubectl describe role pod-reader -n dev
```

---

## Check User Permissions

```bash
kubectl auth can-i get pods
```

---

## Check User Namespace Permissions

```bash
kubectl auth can-i create deployments -n dev
```

---

## View aws-auth ConfigMap

```bash
kubectl describe configmap aws-auth -n kube-system
```

---

# 12. Interview Questions

## Q1: What is RBAC?

RBAC controls:
- User permissions
- Access management in Kubernetes

---

## Q2: Difference between Role and ClusterRole?

| Role | ClusterRole |
|---|---|
| Namespace scoped | Cluster scoped |
| Single namespace | All namespaces |

---

## Q3: Difference between RoleBinding and ClusterRoleBinding?

| RoleBinding | ClusterRoleBinding |
|---|---|
| Namespace access | Cluster-wide access |

---

## Q4: What is aws-auth ConfigMap?

Used in EKS to:
- Map IAM users/roles into Kubernetes RBAC

---

## Q5: How to check permissions?

```bash
kubectl auth can-i <verb> <resource>
```

---

# 13. Summary

- RBAC secures Kubernetes access
- Role = Namespace permissions
- ClusterRole = Cluster-wide permissions
- RoleBinding = Bind Role
- ClusterRoleBinding = Bind ClusterRole
- aws-auth maps IAM users in EKS
- RBAC improves cluster security
- Principle of least privilege should be followed

---

# 📂 Suggested Folder Structure

```bash
rbac/
│
├── role.yml
├── role-bind.yml
├── cluster-role.yml
├── cluster-role-binding.yml
├── auth.yml
└── README.md
```

---

# 🚀 Best Practices

- Use least privilege access
- Avoid cluster-admin for everyone
- Separate developer and admin permissions
- Use namespaces properly
- Audit RBAC regularly

---

<div align="center">

# 🔐 Secure Kubernetes with RBAC ☸️

### Fine-Grained Access Control for Production Clusters

</div>
