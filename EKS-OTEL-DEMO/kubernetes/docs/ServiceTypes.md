# Kubernetes Service Types

Kubernetes Services define how applications running in Pods are exposed and accessed within or outside the cluster. Each Service type serves a different networking purpose.

---

## 1. ClusterIP

### Description

- Default Kubernetes Service type
- Exposes the Service on an **internal cluster IP**
- Accessible only **within the cluster**

---

### Use Case

ClusterIP is used for **internal communication** between components.

Typical scenarios:

- Microservice-to-microservice communication
- Backend APIs
- Internal databases

---

### How It Works

- Kubernetes assigns a **stable virtual IP**
- This IP is reachable only from within the cluster
- Traffic is load-balanced across matching Pods

---

### Key Idea

```text id="clusterip_flow"
Pod → ClusterIP Service → Selected Pods
```

---

## 2. NodePort

### Description

- Exposes the Service on a **static port on each node**
- Accessible using:

```text id="nodeport_format"
<NodeIP>:<NodePort>
```

---

### Use Case

NodePort is commonly used for:

- Development environments
- Simple external access without cloud load balancer
- Testing deployments

---

### How It Works

- Kubernetes assigns a port from a predefined range:

```text id="nodeport_range"
30000 - 32767
```

- This port is opened on every node
- Traffic is forwarded to the Service and then to Pods

---

### Key Idea

```text id="nodeport_flow"
External Client → NodeIP:NodePort → Service → Pods
```

---

## 3. LoadBalancer

### Description

- Exposes the Service externally using a **cloud provider load balancer**
- Automatically provisions an external IP

---

### Use Case

Best suited for:

- Production applications
- Internet-facing services
- High availability workloads

---

### How It Works

- Kubernetes requests a Load Balancer from the cloud provider (AWS, GCP, Azure)
- The load balancer gets a public IP/DNS
- Traffic is distributed across backend Pods

---

### Key Idea

```text id="lb_flow"
Internet → Cloud Load Balancer → Kubernetes Service → Pods
```

---

## Comparison Summary

| Type | Access Scope | Use Case |
|------|--------------|----------|
| ClusterIP | Internal only | Microservices, internal APIs |
| NodePort | External via node IP | Testing, dev environments |
| LoadBalancer | External via cloud LB | Production web apps |

---

## Conclusion

Choosing the correct Service type depends on how your application needs to be accessed:

- **ClusterIP** → internal communication  
- **NodePort** → simple external testing  
- **LoadBalancer** → production-grade external access  

In real-world Kubernetes setups, you often use a **combination of all three** depending on the architecture.
```
