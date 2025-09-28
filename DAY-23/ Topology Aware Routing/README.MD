---

# 🗺️ Kubernetes Topology Aware Routing Example

Topology Aware Routing in Kubernetes ensures service traffic is routed to pods **within the same zone** whenever possible.
This improves **latency**, **reliability**, and reduces **cross-zone network costs**. 🌐⚡

---

## 🛍️ Real-Life Analogy

Imagine a **shopping app** with servers deployed in 3 zones: **Zone A**, **Zone B**, and **Zone C**.

When a user connects from **Zone A**, the app should serve them from **backend servers in Zone A**.

✅ **Benefits**:

* Faster response time ⏱️
* Reduced cross-zone network charges 💰

---

## 📋 Prerequisites

* Kubernetes **v1.23+** (v1.33 used in this example)
* Multi-zone cluster (at least 2 zones) – in this example: `ap-south-1a` & `ap-south-1b`
* `kubectl` configured for the cluster

### 🔍 Verify Multi-Zone Cluster

```bash
kubectl get nodes --show-labels


```

Look for labels like:

```
topology.kubernetes.io/zone
topology.kubernetes.io/region
```

---

## 🏗️ Step 1: Create Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 8
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app: my-app
      containers:
      - name: my-app
        image: nginx:latest
        ports:
        - containerPort: 80
```

Apply the deployment:

```bash
kubectl apply -f my-app-deployment.yaml
```

---

## 🛠️ Step 2: Create Service with Topology Aware Routing

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  annotations:
    service.kubernetes.io/topology-mode: Auto
spec:
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

Apply the service:

```bash
kubectl apply -f my-app-service.yaml
```

---

## 👀 Step 3: Verify Pod Placement

Check pod distribution across zones:

```bash
kubectl get pods -o wide -l app=my-app
```

* Pods should be **evenly spread across zones** 🌍
* Note which pod is on which node/zone 🏷️

---

## 🔎 Step 4: Inspect EndpointSlices

```bash
kubectl get endpointslice -l kubernetes.io/service-name=my-service -o yaml
```

You should see:

```yaml
hints:
  forZones:
    - name: ap-south-1a
    - name: ap-south-1b
```

✅ This confirms that **kube-proxy prefers same-zone pods**.

---

## 🧪 Step 5: Test Traffic Routing

### 5.1 Create Test Pods in Each Zone

```bash
kubectl run tester-a --image=busybox:1.36 --restart=Never -it -- /bin/sh
# Deployed on node in ap-south-1a

kubectl run tester-b --image=busybox:1.36 --restart=Never -it -- /bin/sh
# Deployed on node in ap-south-1b
```

### 5.2 Send Requests

Inside `tester-a` (Zone A) 🌐:

```bash
wget -qO- my-service
```

Inside `tester-b` (Zone B) 🌐:

```bash
wget -qO- my-service
```

Check which pod served the requests:

```bash
kubectl logs -l app=my-app --max-log-requests=10
```

### ✅ Expected Results

* Traffic from **Zone A** hits pods in **Zone A**
* Traffic from **Zone B** hits pods in **Zone B**
* **Logs** and **EndpointSlices** confirm **topology-aware routing** is active

---

## 📈 Result

* Pods are **spread across zones** 🌍
* Service routes traffic **preferentially to same-zone pods** 💡
* Reduced **latency** ⏱️ and **cross-zone network usage** 💰

---

## ⚠️ Notes

* Works best in **multi-zone clusters**
* Ensure `kube-proxy` is up-to-date and supports **Topology Aware Routing**
* Can be combined with **NetworkPolicies** for enhanced traffic control 🔒

---
