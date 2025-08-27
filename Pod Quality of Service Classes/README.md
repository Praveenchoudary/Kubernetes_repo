
````markdown
# 🚀 Kubernetes Pod QoS (Quality of Service)  

Kubernetes uses **Quality of Service (QoS) classes** to decide how Pods get resources and which Pods are **evicted first** when nodes run out of CPU/Memory.  

There are **3 QoS classes**:  
- ✅ **Guaranteed**  
- ⚡ **Burstable**  
- 🟢 **BestEffort**  

---

## 🔹 1. Guaranteed QoS ✅  
👉 Like a **confirmed flight ticket ✈️** – nobody can take your seat.  

A Pod is **Guaranteed** if:  
- Every container has CPU & Memory **requests = limits**.  

📌 **Use Case:** Critical apps (e.g., payment services, databases).  

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: payment-service
spec:
  containers:
  - name: payment-api
    image: nginx
    resources:
      requests:
        cpu: "1000m"
        memory: "512Mi"
      limits:
        cpu: "1000m"
        memory: "512Mi"
````

✅ Always gets resources
✅ Highest priority
✅ Last to be evicted

<img width="1203" height="434" alt="image" src="https://github.com/user-attachments/assets/83d4fb6b-ff2f-4418-8a8a-58b82491d4c9" />

---

## 🔹 2. Burstable QoS ⚡

👉 Like a **train with a reserved seat 🚆** – you have a guaranteed spot, but can also use extra space if free.

A Pod is **Burstable** if:

* At least one container has requests set.
* Requests ≠ Limits.

📌 **Use Case:** Web servers, APIs that need baseline resources but handle bursts.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  containers:
  - name: frontend
    image: httpd
    resources:
      requests:
        cpu: "500m"
        memory: "256Mi"
      limits:
        cpu: "1"
        memory: "512Mi"
```

⚡ Gets minimum 0.5 CPU + 256Mi RAM
⚡ Can burst to 1 CPU + 512Mi RAM
⚡ Evicted **after BestEffort, before Guaranteed**

<img width="1203" height="434" alt="image" src="https://github.com/user-attachments/assets/3e3fa4f1-2bf5-46fc-8ff0-4de154787c4b" />


---

## 🔹 3. BestEffort QoS 🟢

👉 Like a **standby passenger 🧳** – you fly only if there are empty seats.

A Pod is **BestEffort** if:

* No requests/limits are set.

📌 **Use Case:** Background tasks (logs, batch jobs, demo apps).

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: log-collector
spec:
  containers:
  - name: logger
    image: nginx
```

🟢 Runs only if free resources exist
🟢 Lowest priority
🟢 First to be killed when node has pressure

<img width="1203" height="434" alt="image" src="https://github.com/user-attachments/assets/81e46c13-192f-4238-bbe2-5ab1157b638d" />

---


## 🔍 Checking QoS Class

Run the command:

```bash
kubectl get pod payment-service -o jsonpath='{.status.qosClass}'
```

Possible outputs:

* `Guaranteed`
* `Burstable`
* `BestEffort`

  Example:  I kept pressure on the nodes by increase the stress At this time my two pods Burstable and Besteffort went down and only Guratted is running

  <img width="1226" height="170" alt="image" src="https://github.com/user-attachments/assets/b07fa3bc-ccb4-43f9-a494-e24abc2aadfd" />

---

## 🎯 Quick Comparison

| **QoS Class** | **Analogy**                | **Use Case**                        | **Eviction Priority** |
| ------------- | -------------------------- | ----------------------------------- | --------------------- |
| ✅ Guaranteed  | Confirmed flight ticket ✈️ | Critical apps (payments, databases) | Last                  |
| ⚡ Burstable   | Reserved seat in train 🚆  | Web apps, APIs                      | Medium                |
| 🟢 BestEffort | Standby passenger 🧳       | Logs, batch jobs                    | First                 |

