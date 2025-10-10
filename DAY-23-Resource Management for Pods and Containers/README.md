
# Kubernetes Resource Management for Pods and Containers

Managing resources efficiently in Kubernetes ensures fair usage, prevents resource contention, and maintains cluster stability. Kubernetes provides mechanisms to enforce resource limits both at the **namespace level** and **container/pod level**.

This guide covers:

- Resource Quotas
- Limit Ranges
- Hands-on examples with real-life scenarios

---

## **1. Kubernetes Resource Quotas**

**ResourceQuota** allows administrators to set constraints on the **total resource usage per namespace**.  
This ensures that no single team or application monopolizes cluster resources.

### **Real-life Scenario**
Imagine a cluster shared by multiple teams:

- **Team A:** Running multiple web services  
- **Team B:** Running batch jobs  

Without quotas, Team B's heavy batch jobs could consume all CPU and memory, impacting Team A's services. ResourceQuota ensures fair resource distribution.

### **Example: ResourceQuota**

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: myrs
  namespace: dev
spec:
  hard:
    requests.cpu: "150m"      # Total requested CPU allowed
    requests.memory: "12Mi"   # Total requested memory allowed
    limits.cpu: "200m"        # Total CPU limit allowed
    limits.memory: "38Mi"     # Total memory limit allowed
````

Apply the quota:

```bash
kubectl apply -f resource-quota.yaml
```

---

### **Step 1: Create Namespace**

```bash
kubectl create namespace dev
```

### **Step 2: Create a Pod within the Quota**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  namespace: dev
spec:
  containers:
  - name: cont1
    image: nginx
    ports:
    - containerPort: 80
    resources:
      requests:
        cpu: "30m"
        memory: "2Mi"
      limits:
        cpu: "50m"
        memory: "4Mi"
```

**Pod Consumption from Quota:**

| Resource | Request | Limit |
| -------- | ------- | ----- |
| CPU      | 30m     | 50m   |
| Memory   | 2Mi     | 4Mi   |

---

### **Step 3: Attempt to Create a Second Pod**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod2
  namespace: dev
spec:
  containers:
  - name: cont2
    image: redis
    resources:
      requests:
        cpu: "100m"
        memory: "12Mi"
      limits:
        cpu: "250m"
        memory: "38Mi"
```

* Total memory request after pod1 + pod2 = 2Mi + 12Mi = 14Mi
* Quota allows only **12Mi** → Kubernetes rejects pod2

**Error Example:**

```
Error from server (Forbidden): exceeded quota: myrs, requested: memory=12Mi, used: memory=2Mi, limited: memory=12Mi
```

---

## **2. Kubernetes Limit Ranges**

**LimitRange** defines **minimum, maximum, and default resource requests/limits** for Pods and Containers in a namespace.

### **Real-life Scenario**

If developers forget to define resource requests in their Pod manifests:

* Some Pods may consume excessive CPU/memory
* Other Pods may starve due to resource contention

LimitRange automatically enforces safe defaults.

### **Example: LimitRange**

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: limit-range-example
  namespace: dev
spec:
  limits:
  - type: Pod
    max:
      cpu: "2"
      memory: "4Gi"
    min:
      cpu: "200m"
      memory: "256Mi"
    maxLimitRequestRatio:
      cpu: "4"
      memory: "8"
  - type: Container
    default:
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "250m"
      memory: "256Mi"
    max:
      cpu: "1"
      memory: "1Gi"
    min:
      cpu: "100m"
      memory: "128Mi"
    maxLimitRequestRatio:
      cpu: "2"
      memory: "4"
  - type: PersistentVolumeClaim
    max:
      storage: "10Gi"
    min:
      storage: "1Gi"
    default:
      storage: "5Gi"
    defaultRequest:
      storage: "2Gi"
    maxLimitRequestRatio:
      storage: "2"
```

* **Pod:** CPU 200m–2, Memory 256Mi–4Gi
* **Container:** Default CPU 250m, Limit 500m; Memory default 256Mi, Limit 512Mi
* **PVC:** Storage default 2Gi, Limit 5Gi, max 10Gi

Apply the LimitRange:

```bash
kubectl apply -f limit-range.yaml
```

### **Step 4: Create a Pod without specifying resources**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod3
  namespace: dev
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
```

* Kubernetes automatically applies **default requests and limits** from LimitRange
* Ensures predictable resource usage and prevents overconsumption

---

## ✅ **Key Takeaways**

* **Requests:** Resources guaranteed for the Pod
* **Limits:** Maximum resources a Pod/container can use
* **ResourceQuota:** Controls **total usage per namespace**
* **LimitRange:** Enforces **default, min, and max values** for Pods and containers
* Kubernetes prevents Pods from exceeding quotas → Ensures **fair resource allocation**

---

Do you want me to do that next?
```
