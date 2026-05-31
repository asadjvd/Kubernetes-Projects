# Kubernetes Services

## What is a Kubernetes Service?

A **Kubernetes Service** is an abstraction that defines a logical set of Pods and a stable way to access them.

Since Pods are **ephemeral (temporary)** and can change frequently, Services provide a consistent interface to reach them.

A Service typically provides:

- Stable IP address
- DNS name
- Load balancing across Pods

---

## Why Kubernetes Services Are Important

In Kubernetes, Pods are constantly created, destroyed, or rescheduled. This causes their IP addresses to change.

Services solve this problem by acting as a stable communication layer.

---

## How Services Help with Service Discovery

Kubernetes Services simplify communication between components in a cluster.

### 1. Stable Endpoints

Each Service provides:

- A fixed Cluster IP
- A DNS name inside the cluster

This remains unchanged even if Pods are replaced.

---

### 2. Load Balancing

Services automatically distribute traffic across all healthy Pods.

### Example

If you have:

- Pod A
- Pod B
- Pod C

A Service will distribute traffic among them evenly.

---

### 3. Service Discovery

Kubernetes provides built-in service discovery via:

- DNS (recommended)
- Environment variables (legacy approach)

Example DNS format:

```text
my-service.default.svc.cluster.local
```

---

## Types of Kubernetes Services

---

### 1. ClusterIP (Default)

Exposes the Service only inside the cluster.

- Internal communication only
- Not accessible from outside the cluster

### Use Case

- Backend services
- Internal APIs

---

### 2. NodePort

Exposes the Service on a static port on each node.

- Accessible from outside the cluster
- Uses node IP + port

### Use Case

- Simple external access
- Testing and development

---

### 3. LoadBalancer

Exposes the Service externally using a cloud provider load balancer.

- Automatically provisions external load balancer (AWS, GCP, Azure)
- Production-grade external access

### Use Case

- Public web applications
- Production APIs

---

### 4. ExternalName

Maps a Service to an external DNS name.

- No proxying or load balancing
- Just DNS redirection

### Use Case

- External databases
- Third-party APIs

---

## Example Kubernetes Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
  type: ClusterIP
```

---

## Explanation

### Selector

```yaml
selector:
  app: MyApp
```

This connects the Service to Pods with label:

```text
app: MyApp
```

---

### Ports

| Field | Meaning |
|------|--------|
| `port` | Service port (inside cluster) |
| `targetPort` | Container port inside Pod |

---

### Type

```yaml
type: ClusterIP
```

Defines that this Service is internal-only.

---

## Traffic Flow

```text
Client Pod
    ↓
Kubernetes Service (Stable IP/DNS)
    ↓
Load Balancer (inside cluster)
    ↓
Matching Pods (via selector)
```

---

## Key Benefits

Kubernetes Services provide:

- 🔐 Stable networking for Pods  
- ⚖️ Built-in load balancing  
- 🔎 Automatic service discovery  
- 🌐 Multiple exposure options (internal/external)  

---

## Summary

Without Services:

- Pods communicate using unstable IPs
- Manual tracking is required

With Services:

- Communication becomes stable
- Scaling becomes seamless
- Applications become production-ready
```
