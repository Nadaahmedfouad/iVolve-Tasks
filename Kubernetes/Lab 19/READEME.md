# Lab 19 – Kubernetes RBAC and Service Accounts

## Objective

The goal of this lab was to secure Kubernetes access using RBAC (Role-Based Access Control) and Service Accounts.

I configured a Jenkins ServiceAccount with limited permissions to manage Deployments inside the `ivolve` namespace.

## What I Did

1. Created a ServiceAccount named `jenkinssa` in the `ivolve` namespace.
2. Generated a temporary authentication token using `kubectl create token`.
3. Created a Role named `ivolvedeployrole` allowing only `create` and `update` operations on Deployments.
4. Created a RoleBinding named `ivolvedeployrolebinding` to assign the Role to the Jenkins ServiceAccount.
5. Tested the permissions using `kubectl auth can-i`.

## RBAC Configuration

The Role was configured with the following permissions:

```yaml
rules:
  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["create", "update"]
```

These permissions apply only within the `ivolve` namespace.

## Verification

I used `kubectl auth can-i` to check what the Jenkins ServiceAccount could access.

| Action                                  | Expected Result |
| --------------------------------------- | --------------- |
| Create Deployments in ivolve            | Allowed         |
| Update Deployments in ivolve            | Allowed         |
| Delete Deployments in ivolve            | Denied          |
| Read Secrets in ivolve                  | Denied          |
| Create Deployments in default namespace | Denied          |

This demonstrates how RBAC can restrict a ServiceAccount to specific resources and operations.

## Screenshots

- ServiceAccount creation and verification.
- Role and RoleBinding configuration.
- Allowed and denied permission tests.

## What I Learned

A ServiceAccount provides an identity for applications interacting with Kubernetes.

A Role defines what operations are allowed, while a RoleBinding assigns those permissions to a specific identity.

RBAC helps enforce the principle of least privilege by giving applications only the permissions they need.
