
---

# 🚀 Kubernetes Gateway API with Envoy Gateway

### Production-Ready Implementation on E2E Kubernetes Cluster

---

## 📌 Overview

As Kubernetes workloads grow in scale and complexity, the traditional **Ingress** model becomes difficult to manage and unsafe for multi-team environments.

To solve these challenges, Kubernetes introduced the **Gateway API**, a modern, extensible, and role-oriented networking API.

This repository demonstrates a **complete, production-grade Gateway API setup** using **Envoy Gateway** on an **E2E Kubernetes Cluster**.

---

## 🎯 What This Guide Covers

* Gateway API fundamentals
* North–South vs East–West traffic
* Envoy Gateway installation
* HTTP & HTTPS Gateway
* TLS automation using cert-manager + Let’s Encrypt
* Host-based routing
* URL rewrite
* Traffic splitting (50/50)
* Weighted canary deployment (80/20)

---

## 🚦 Traffic Types in Kubernetes

### 🌍 North–South Traffic

Traffic entering or leaving the cluster.

**Examples**

* Browser accessing a website
* External API calls

👉 Handled by **Ingress / Gateway API**

---

### 🔁 East–West Traffic

Traffic within the cluster.

**Examples**

* Frontend → Backend
* Order → Payment service

👉 Handled by **Service Mesh**

> ⚠️ Gateway API is **not** for internal service-to-service traffic.

---

## 🧱 Gateway API Core Components

| Component    | Purpose                        | Managed By    |
| ------------ | ------------------------------ | ------------- |
| GatewayClass | Selects gateway implementation | Platform Team |
| Gateway      | Entry point (ports, TLS)       | Platform/Ops  |
| HTTPRoute    | Routing rules                  | App Team      |

---

## 🛠 Prerequisites

* Kubernetes cluster (E2E)
* `kubectl` configured
* Helm installed
  [https://helm.sh/docs/intro/install/](https://helm.sh/docs/intro/install/)
* Public domain name (GoDaddy used here)

---

## 🪜 Step-by-Step Implementation

---

## 🔹 Step 1: Create Kubernetes Cluster on E2E

Create a Kubernetes cluster using the **E2E Cloud Console**.

---

## 🔹 Step 2: Install Gateway API CRDs

```bash
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.1.0/standard-install.yaml
```

Verify:

```bash
kubectl get crds | grep gateway
```

---

## 🔹 Step 3: Install Envoy Gateway Controller

```bash
helm install eg oci://docker.io/envoyproxy/gateway-helm \
  --version v1.6.1 \
  -n envoy-gateway-system \
  --create-namespace
```

Wait for readiness:

```bash
kubectl wait --timeout=5m -n envoy-gateway-system \
deployment/envoy-gateway --for=condition=Available
```

---

## 🔹 Step 4: Create GatewayClass

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: envoy-gateway-class
spec:
  controllerName: gateway.envoyproxy.io/gatewayclass-controller
```

```bash
kubectl apply -f gatewayclass.yaml
```

---

## 🔹 Step 5: Create Gateway (HTTP + HTTPS)

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: api-gateway
spec:
  gatewayClassName: envoy-gateway-class
  listeners:
    - name: http
      protocol: HTTP
      port: 80
    - name: https
      protocol: HTTPS
      port: 443
      tls:
        mode: Terminate
        certificateRefs:
          - name: apigateway-tls
```

```bash
kubectl apply -f gateway.yaml
kubectl get gateway api-gateway
```

---

## 🌐 Step 6: Domain Mapping (GoDaddy)

Create an **A record**:

| Field | Value                   |
| ----- | ----------------------- |
| Name  | apigateway              |
| Type  | A                       |
| Value | `<Gateway External IP>` |

Verify:

```bash
dig apigateway.praveens.online
```

---

## 🔐 Step 7: Install cert-manager

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.5/cert-manager.yaml
```

Verify:

```bash
kubectl get pods -n cert-manager
```

---

## 🔑 Step 8: Configure Let’s Encrypt ClusterIssuer

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    email: admin@praveens.online
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-prod-key
    solvers:
      - http01:
          gatewayHTTPRoute:
            parentRefs:
              - name: api-gateway
```

---

## 📜 Step 9: Request TLS Certificate

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: apigateway-cert
spec:
  secretName: apigateway-tls
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  dnsNames:
    - apigateway.praveens.online
```

---

## 🚀 Step 10: Routing Examples

---

### 🌍 Host-Based Routing

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: host-route
spec:
  parentRefs:
    - name: api-gateway
  hostnames:
    - apigateway.praveens.online
  rules:
    - backendRefs:
        - name: backend-1
          port: 80
```

---

### 🔁 URL Rewrite (`/app → /`)

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: rewrite-example
spec:
  parentRefs:
    - name: api-gateway
  hostnames:
    - apigateway.praveens.online
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /app
      filters:
        - type: URLRewrite
          urlRewrite:
            path:
              type: ReplacePrefixMatch
              replacePrefixMatch: /
      backendRefs:
        - name: backend-1
          port: 80
```

---

### ⚖️ Traffic Splitting (50/50)

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: traffic-splitting
spec:
  parentRefs:
    - name: api-gateway
  hostnames:
    - apigateway.praveens.online
  rules:
    - backendRefs:
        - name: backend-1
          port: 80
        - name: backend-2
          port: 80
```

---

### 🐤 Weighted Canary Deployment (80/20)

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: weighted-canary
spec:
  parentRefs:
    - name: api-gateway
  hostnames:
    - apigateway.praveens.online
  rules:
    - backendRefs:
        - name: backend-1
          port: 80
          weight: 80
        - name: backend-2
          port: 80
          weight: 20
```

---

## 🧪 Browser Verification

Open:

```
https://apigateway.praveens.online
```

Refresh 20–30 times:

* Mostly → backend-1
* Occasionally → backend-2

✅ Canary deployment confirmed

---

## ✨ Additional Gateway API Features

* Header-based routing
* Request mirroring
* Rate limiting
* TCP / UDP routing

📘 Reference:
[https://gateway.envoyproxy.io/docs/tasks/traffic/](https://gateway.envoyproxy.io/docs/tasks/traffic/)

---

## 🏁 Conclusion

Gateway API provides:

* Clear role separation
* Safer production networking
* Advanced traffic control
* Controller-agnostic behavior

🚀 **Recommended replacement for Ingress in modern Kubernetes environments**

---

