

![image](https://github.com/user-attachments/assets/13eee7c2-fa58-4ec8-aecb-9f0f39e9f43f)

# 🌐 Kubernetes Network Policies 🚀

Kubernetes Network Policies provide a robust mechanism to control the communication between pods and other network endpoints. They are essential for securing your applications by defining rules that specify how pods can communicate with each other and other network resources. In this hands-on guide, we will cover the fundamentals and demonstrate how to implement network policies with a practical example involving frontend and backend applications.


## 1. 📝 Introduction to Kubernetes Network Policies

Network Policies in Kubernetes are a crucial tool for securing your applications. They allow you to define rules that control how groups of pods can communicate with each other and other network resources. This is essential for isolating sensitive workloads and ensuring that only authorized traffic is allowed within your cluster. 🔒

## 2. ⚙️ Prerequisites

Before diving into Kubernetes Network Policies, ensure you have the following:

- A basic understanding of Kubernetes concepts. 💡
- kubectl configured to interact with your cluster. 🖥️
- A running Kubernetes cluster. 🚀

## 3. 🔍 Understanding Network Policies

Network Policies are implemented using Kubernetes resources that define how pods are allowed to communicate with each other and other network endpoints. They use labels to select pods and define rules for ingress (incoming) and egress (outgoing) traffic.

### Key Concepts: 📌

- **Pod Selector**: Selects the group of pods to which the policy applies. 🏷️
- **Ingress Rules**: Define allowed incoming traffic. 🚪
- **Egress Rules**: Define allowed outgoing traffic. 🌐
- **Policy Types**: Can be either ingress, egress, or both. 🔄

- Got it 👍 You want to make a **GitHub README file** that explains **Kubernetes Network Policies** in detail (theory + real-life use cases + real-time examples + implementation steps on EKS with Calico).

Here’s a complete **README.md** draft you can use directly in your repo:

---

# 🚀 Kubernetes Network Policies with Real-Life Examples

## 📌 Introduction

Kubernetes **Network Policies** are a way to **control network traffic** between pods, namespaces, and external endpoints.
They work like a firewall for pods, allowing you to define **who can talk to whom** inside your cluster.

By default in Kubernetes:

* **All pods can talk to all other pods.**
* Once you apply a **NetworkPolicy**, traffic is **DENIED by default** unless explicitly allowed.

👉 Network Policies are supported only if your cluster has a **CNI plugin** that enforces them (e.g., **Calico, Cilium, Weave Net**).
On **AWS EKS**, you need to enable **Calico** to use them.

---

## 🔑 Key Concepts

* **Pod Selector** → Selects the pods to which the policy applies.
* **Ingress Rules** → Controls **incoming** traffic.
* **Egress Rules** → Controls **outgoing** traffic.
* **Policy Types** → Can be `Ingress`, `Egress`, or both.

---

## ✅ Real-Life Use Cases

1. **E-commerce Application**

   * Only **frontend** pods should talk to **backend** pods.
   * Database pods should only allow traffic from backend pods.
   * No other pod should directly access the DB.

2. **Multi-Tenant SaaS Application**

   * Each tenant runs in a separate namespace.
   * NetworkPolicy ensures **isolation** → one tenant cannot access another tenant’s services.

3. **Financial / Banking Applications**

   * Outgoing traffic (egress) from sensitive workloads restricted → No pod should make random external calls.

4. **Healthcare Apps**

   * Policies ensure only whitelisted microservices can talk to protected services like **patient records DB**.

---

## ⚙️ Enabling Calico on EKS

If you created EKS using Terraform, you can install **Calico** as an add-on:

```bash
# Install Calico on EKS
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

# Verify pods
kubectl get pods -n kube-system | grep calico
```

You should see `calico-node` pods running.

---

## 🛠️ Hands-On Example

### 1️⃣ Deploy Frontend & Backend Apps

`deployments.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-app
  labels:
    app: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-app
  labels:
    app: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

Apply:

```bash
kubectl apply -f deployments.yaml
```

---

### 2️⃣ Create a Network Policy

`backend-network-policy.yaml`

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-network-policy
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: frontend
```

Apply:

```bash
kubectl apply -f backend-network-policy.yaml
```

---

### 3️⃣ Verify Connectivity

```bash
# Get frontend pod
FRONTEND_POD=$(kubectl get pods -l app=frontend -o jsonpath='{.items[0].metadata.name}')

# Get backend pod
BACKEND_POD=$(kubectl get pods -l app=backend -o jsonpath='{.items[0].metadata.name}')
BACKEND_IP=$(kubectl get pod $BACKEND_POD -o jsonpath='{.status.podIP}')

# ✅ Allowed: Frontend → Backend
kubectl exec -it $FRONTEND_POD -- curl -s $BACKEND_IP

# ❌ Blocked: Random pod → Backend
kubectl run busybox --rm -it --image=busybox -- /bin/sh
wget --spider --timeout=1 $BACKEND_IP
```

---

## 📘 More Example Policies

### 1. Deny All Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

### 2. Allow Only HTTPS Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-https
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
  - Ingress
  ingress:
  - ports:
    - protocol: TCP
      port: 443
```

### 3. Restrict Egress to Specific IP Range

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-egress
spec:
  podSelector:
    matchLabels:
      app: secure-app
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 192.168.1.0/24
```

---

## 🛡️ Real-Time Example in Production

Imagine an **online banking application**:

* **Frontend Pods** → Accessible from the internet.
* **Backend Pods** → Only accessible from frontend pods.
* **Database Pods** → Only accessible from backend pods.
* **Egress Restricted** → No pod should call unknown external services (prevents data leaks).

👉 This setup follows the principle of **Zero Trust Networking** inside Kubernetes.

---

## 📝 Conclusion

* Network Policies = **firewall rules inside Kubernetes**.
* Helps you **segment traffic**, **secure workloads**, and **comply with regulations**.
* Must have for **production-grade EKS clusters**.

---


