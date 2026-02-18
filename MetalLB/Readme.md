
---

# 🚀 MetalLB with Helm on Kubernetes (Layer 2 Mode)


This repository provides a **complete step-by-step guide** to install and configure **MetalLB** in a Kubernetes cluster running on **on-premise**, **bare-metal**, or **local VM** environments using **Helm**.

It also demonstrates:

* Configuring `IPAddressPool`
* Creating `L2Advertisement`
* Exposing services using **LoadBalancer**
* Using **NGINX Ingress Controller** with MetalLB

---

# 📖 Table of Contents

* Introduction
* About MetalLB
* Prerequisites
* Installing MetalLB with Helm
* Configuring IPAddressPool & L2Advertisement
* Testing LoadBalancer Service
* How MetalLB Works (Layer 2)
* Adding NGINX Ingress Controller
* Conclusion
* Resources

---

# 📘 Introduction

In cloud platforms like AWS, Azure, or GCP, Kubernetes automatically provisions external load balancers.

However, in **bare-metal** or **on-premise** environments:

👉 There is **no cloud provider** to assign external IPs.

This results in:

```
EXTERNAL-IP: <pending>
```

MetalLB solves this problem by providing **software-based LoadBalancer functionality**.

---

## ❌ LoadBalancer Without MetalLB

```
kubectl get svc my-service
```

```
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)
my-service   LoadBalancer   10.xx.xx.xx     <pending>     80/TCP
```

Why?

Kubernetes expects a cloud provider to allocate an external IP.

---

## ✅ LoadBalancer With MetalLB

```
kubectl get svc my-service
```

```
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)
my-service   LoadBalancer   10.xx.xx.xx     192.168.1.100    80/TCP
```

MetalLB:

* Watches LoadBalancer services
* Allocates IP from pool
* Announces IP using Layer 2 (ARP)

---

# 📦 About MetalLB

MetalLB is a load balancer implementation for bare-metal Kubernetes clusters.

### Features

* Assigns external IPs
* Works without cloud provider
* Supports Layer2 and BGP modes
* Free & open source

---

## ☁️ Cloud Provider vs MetalLB

| Feature            | Cloud Provider | MetalLB |
| ------------------ | -------------- | ------- |
| IP Allocation      | Automatic      | Manual  |
| Cost               | Paid           | Free    |
| Cloud Dependency   | Yes            | No      |
| Bare-metal Support | ❌              | ✅       |

---

# ⚙️ Prerequisites

Make sure you have:

* Kubernetes Cluster (Kubespray / Bare-metal / Vagrant)
* kubectl installed
* Helm installed
* Layer2 compatible network
* VirtualBox & Vagrant (optional)

---

# 🧩 Installing MetalLB with Helm

## 1️⃣ Add Helm Repository

```
helm repo add metallb https://metallb.github.io/metallb
```

## 2️⃣ Update Repo

```
helm repo update
```

## 3️⃣ Create Namespace

```
kubectl create namespace metallb-system
```

## 4️⃣ Install MetalLB

```
helm install metallb metallb/metallb --namespace metallb-system
```

## 5️⃣ Verify Pods

```
kubectl get pods -n metallb-system
```

Components:

| Component  | Role                 |
| ---------- | -------------------- |
| controller | Assigns external IP  |
| speaker    | Announces IP via ARP |

---

# 🌐 Configuring IPAddressPool & L2Advertisement

Create file:

```
metallb-config.yaml
```

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.59.100-192.168.59.120
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: l2-adv
  namespace: metallb-system
spec:
  ipAddressPools:
  - first-pool
```

Apply:

```
kubectl apply -f metallb-config.yaml
```

---

## ⚠️ Important: Enable Strict ARP

If using **kube-proxy IPVS**:

```
kube_proxy_mode: ipvs
kube_proxy_strict_arp: true
```

Reapply Kubespray after updating config.

---

# 🧪 Testing LoadBalancer Service

## Deploy Test App

```
kubectl create deployment nginx-test --image=nginx
```

## Expose as LoadBalancer

```
kubectl expose deployment nginx-test --port=80 --type=LoadBalancer
```

Check:

```
kubectl get svc
```

Example:

```
EXTERNAL-IP: 192.168.59.100
```

Test:

```
curl http://192.168.59.100
```

---

# ⚙️ How MetalLB Works (Layer 2)

Flow:

1. Controller assigns IP from pool
2. Speaker announces IP via ARP
3. Network routes traffic to node
4. kube-proxy forwards traffic to pods

### Roles

* MetalLB → IP allocation + announcement
* kube-proxy → Load balancing

---

## kube-proxy Modes

| Mode     | Description      |
| -------- | ---------------- |
| iptables | Simple rules     |
| IPVS     | High performance |

---

# 🌍 Adding NGINX Ingress Controller

## Add Helm Repo

```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

## Create Namespace

```
kubectl create namespace ingress-nginx
```

## Install Controller

```
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --set controller.service.type=LoadBalancer
```

Check service:

```
kubectl get svc -n ingress-nginx
```

MetalLB assigns external IP automatically.

---

## Deploy Sample App

```
kubectl create deployment web-app --image=nginx
kubectl expose deployment web-app --port=80
```

---

## Create Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-app-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: web-app.local
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

```
kubectl apply -f web-app-ingress.yaml
```

---

## Update Hosts File

```
192.168.59.100 web-app.local
```

Test:

```
curl http://web-app.local
```

---

# ✅ Conclusion

You successfully learned how to:

* Install MetalLB using Helm
* Configure IPAddressPool
* Enable Layer2 advertisement
* Expose LoadBalancer services
* Integrate NGINX Ingress

MetalLB enables cloud-like LoadBalancer behavior in bare-metal Kubernetes clusters.

---

# 📚 Resources

* MetalLB Docs → [https://metallb.universe.tf/](https://metallb.universe.tf/)
* Kubernetes → [https://kubernetes.io/](https://kubernetes.io/)
* Helm → [https://helm.sh/](https://helm.sh/)

---


