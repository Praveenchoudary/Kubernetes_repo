
---

# 📄 `INGRESS-NGINX-WITH-METALLB.md`

---

# 🚀 NGINX Ingress Controller with MetalLB (Bare-Metal Kubernetes)

This guide explains how to install and configure the **NGINX Ingress Controller** in a Kubernetes cluster using **MetalLB LoadBalancer**.

It also demonstrates a common issue with `externalTrafficPolicy: Local`, why it happens, and how to fix it.

---

# 📌 Table of Contents

* Introduction
* Prerequisites
* Install Ingress Controller
* Deploy Sample Application
* Create Ingress Resource
* Reproducing the Issue
* Root Cause Explanation
* Fixing the Issue
* Production Best Practices (Multi-Node Clusters)

---

# 📖 Introduction

In bare-metal Kubernetes environments, MetalLB provides external IPs for services of type `LoadBalancer`.

The NGINX Ingress Controller allows HTTP/HTTPS routing using hostnames such as:

```
myapp.praveens.online
```

---

# ✅ Prerequisites

* Running Kubernetes Cluster
* MetalLB Installed & Configured
* Domain DNS pointing to MetalLB IP
* kubectl configured

---

# 🚀 Step 1 — Install NGINX Ingress Controller

Install using official static manifest:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.6/deploy/static/provider/cloud/deploy.yaml
```

---

## Verify Pods

```bash
kubectl get pods -n ingress-nginx -o wide
```

Example:

```
ingress-nginx-controller-xxxxx   Running   NODE=e2e-83-123
```

---

## Verify Service

```bash
kubectl get svc -n ingress-nginx
```

Example:

```
ingress-nginx-controller   LoadBalancer   164.52.221.190
```

This external IP is assigned by MetalLB.

---

# 📦 Step 2 — Deploy Sample Application

```bash
kubectl create deployment nginx-test --image=nginx
kubectl expose deployment nginx-test --port=80
```

---

# 🌐 Step 3 — Create Ingress Resource

Create `nginx-test-ing.yaml`

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-test-ing
spec:
  rules:
  - host: myapp.praveens.online
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-test
            port:
              number: 80
```

Apply:

```bash
kubectl apply -f nginx-test-ing.yaml
```

---

#  Step 4 — Reproducing the Issue

Check service configuration:

```bash
kubectl get svc ingress-nginx-controller -n ingress-nginx -o yaml | grep externalTrafficPolicy
```

Output:

```
externalTrafficPolicy: Local
```

---

## 🧠 What Happens?

Cluster example:

```
Worker Node      → ingress pod exists
Control-plane    → no ingress pod
```

MetalLB may send traffic to any node.

If traffic hits a node without ingress pod:

```
externalTrafficPolicy = Local
→ kube-proxy cannot find local endpoint
→ Packet is dropped
```

Result:

```
Browser timeout or failed connection
```

---

# 🔍 Root Cause Explanation

`externalTrafficPolicy: Local` means:

> Accept traffic **only if the ingress pod exists on the same node**.

When traffic reaches a node without ingress:

```
iptables rule → DROP traffic
```

---

# ✅ Step 5 — Fix the Issue

Change policy to `Cluster`:

```bash
kubectl patch svc ingress-nginx-controller -n ingress-nginx \
-p '{"spec":{"externalTrafficPolicy":"Cluster"}}'
```

---

## Verify Fix

```bash
kubectl get svc ingress-nginx-controller -n ingress-nginx -o yaml | grep externalTrafficPolicy
```

Output:

```
externalTrafficPolicy: Cluster
```

---

# 🌍 Step 6 — Test Access

Open in browser:

```
http://myapp.praveens.online
```

Expected:

```
Welcome to nginx!
```

---

# ⭐ Production Best Practices (Multiple Worker Nodes)

If your cluster has many worker nodes (example: 10 nodes), consider the following approaches.

---

## ✅ Recommended Approach — Run Ingress on All Nodes

Scale ingress controller:

```bash
kubectl scale deploy ingress-nginx-controller -n ingress-nginx --replicas=10
```

OR use DaemonSet (Best for MetalLB):

```
1 ingress pod per node
```

Benefits:

* No dropped packets
* Real client IP preserved
* High availability

---

## Alternative Approach

Keep:

```
externalTrafficPolicy: Cluster
```

Pros:

* Simple setup
* Works even with single ingress pod

Cons:

* Extra internal routing hop

---

# 📊 Comparison

| Setup                  | Production Ready  |
| ---------------------- | ----------------- |
| Single ingress + Local | ❌ Not recommended |
| Cluster policy         | ✅ Good            |
| Multiple ingress pods  | ⭐ Better          |
| DaemonSet ingress      | ⭐⭐⭐ Best          |

---

# 🏁 Conclusion

Issue occurred because:

```
externalTrafficPolicy: Local
+
Ingress pod running on only one node
+
LoadBalancer sent traffic to another node
→ Traffic dropped by kube-proxy
```

Fix:

```
externalTrafficPolicy: Cluster
```

---

# 🎉 End Result

Your bare-metal Kubernetes cluster now supports:

* MetalLB LoadBalancer
* NGINX Ingress Routing
* Host-based Access via Domain

---



