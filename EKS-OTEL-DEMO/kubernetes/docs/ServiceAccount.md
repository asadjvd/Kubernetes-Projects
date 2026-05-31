# Kubernetes Service Account

## What is a Service Account in Kubernetes?

A **Service Account** in Kubernetes is an identity assigned to **pods** that allows them to interact with the Kubernetes API securely.

Instead of using user credentials, applications running inside pods use Service Accounts to authenticate and perform actions within the cluster.

---

## Why is it Important?

Service Accounts play a key role in securing and managing access inside a Kubernetes cluster.

### 1. Security

Service Accounts allow pods to interact with the Kubernetes API without exposing:

- User credentials
- Admin-level access keys

This reduces the risk of credential leakage.

---

### 2. Granular Access Control

Service Accounts can be bound to specific roles using RBAC (Role-Based Access Control).

This means you can control exactly what a pod can do, such as:

- Read pods only
- Create deployments
- Access specific namespaces

---

### 3. Isolation

Different applications can use different Service Accounts.

### Example

| Application | Service Account |
|-------------|-----------------|
| Frontend API | sa-frontend |
| Backend service | sa-backend |
| Monitoring tool | sa-monitoring |

This ensures each workload only accesses what it needs.

---

## Default Service Account

If no Service Account is specified, Kubernetes automatically assigns the **default Service Account** in the namespace.

### Key Points

- Automatically created per namespace
- Has **minimal permissions**
- Not recommended for production workloads

---

## Example Kubernetes Pod Using a Service Account

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  serviceAccountName: custom-service-account
  containers:
  - name: example-container
    image: example-image
```

---

## Explanation

### `serviceAccountName`

This field defines which Service Account the pod will use.

### Behavior

| Case | Result |
|------|--------|
| `serviceAccountName` specified | Pod uses custom Service Account |
| Not specified | Pod uses default Service Account |

---

## How It Works Internally

```text
Pod → Service Account → Token (JWT) → Kubernetes API Server → Authorization (RBAC)
```

1. Pod gets a Service Account token
2. Token is mounted into the pod
3. Pod uses token to call Kubernetes API
4. RBAC policies determine allowed actions

---

## Best Practices

- Create **dedicated Service Accounts per application**
- Use **least privilege principle (RBAC)**
- Avoid using `default` Service Account in production
- Regularly audit Service Account permissions

---

## Summary

Kubernetes Service Accounts provide:

- 🔐 Secure authentication for pods  
- 🎯 Fine-grained access control via RBAC  
- 🧩 Isolation between workloads  
- 🚫 Safer alternative to shared credentials  

They are a core part of Kubernetes security and should always be configured properly in production environments.
