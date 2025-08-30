
---

# 🚀 Kubernetes Rolling Deployment with `maxSurge` & `maxUnavailable`

This repository demonstrates how **Rolling Deployments** work in Kubernetes with the `RollingUpdate` strategy.
It shows how `maxSurge` and `maxUnavailable` affect the rollout process with **zero downtime**.

---

## 📌 What is a Rolling Deployment?

A **Rolling Deployment** updates Pods **gradually**, replacing old Pods with new ones, while keeping the application available.
Instead of killing all old Pods and then creating new ones, Kubernetes **rolls out** the update in steps.

---

## ⚙️ Strategy Parameters

* **`maxUnavailable`** → Maximum number of Pods that can be **unavailable** during the update.
* **`maxSurge`** → Maximum number of **extra Pods** allowed above the desired replicas during the update.

---

## 📜 Example Deployment

### `nginx-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1   # At most 1 pod can be down
      maxSurge: 1         # At most 1 extra pod can be created
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.19   # Initial version
        ports:
        - containerPort: 80
```

### `nginx-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```

---

## ▶️ Deploy the Application

```bash
kubectl apply -f nginx-deployment.yaml
kubectl apply -f nginx-service.yaml
```

Check rollout status:

```bash
kubectl rollout status deployment/nginx-deployment
```

---

## 🔄 Trigger a Rolling Update

Update image from `nginx:1.19` → `nginx:1.21`:

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.21 --record
```

---

## 📊 Rollout Behavior

### Initial State (4 Pods running v1)

```
Pod1: v1 ✅
Pod2: v1 ✅
Pod3: v1 ✅
Pod4: v1 ✅
```

### Step 1: Surge Pod Created (total = 5 Pods)

```
Pod1: v1 ✅
Pod2: v1 ✅
Pod3: v1 ✅
Pod4: v1 ✅
Pod5: v2 ⏳ (starting)
```

### Step 2: Old Pod Deleted (back to 4 Pods)

```
Pod2: v1 ✅
Pod3: v1 ✅
Pod4: v1 ✅
Pod5: v2 ✅
```

### Final State (all Pods updated to v2)

```
Pod6: v2 ✅
Pod7: v2 ✅
Pod8: v2 ✅
Pod9: v2 ✅
```

<img width="1720" height="642" alt="image" src="https://github.com/user-attachments/assets/dd2457b0-0220-421a-bca5-66e70be96a91" />

✅ The **surge Pod (extra)** is temporary — Kubernetes always ensures the final number of Pods matches `.spec.replicas` (in this case, 4).

---

## 🛑 Rollback (if update fails)

If the new image fails (e.g., typo in image name), Kubernetes keeps old Pods running.
You can rollback safely:

```bash
kubectl rollout undo deployment nginx-deployment
```

---

## 📌 Key Takeaways

* **`maxUnavailable`** → how many Pods can be down during rollout.
* **`maxSurge`** → how many extra Pods can be created temporarily.
* **Final state** always = desired replicas (extra surge Pods are removed).
* Ensures **zero downtime** for production apps.

---

## 🎯 Real-World Use Case

* **E-commerce websites**: use `maxUnavailable: 1` to always keep most Pods available.
* **Exam registration systems**: use `maxSurge: 1` to handle peak load during upgrades.
* **Banking apps**: set both values carefully for **high availability** and **safe rollouts**.

---

👉 You can now run this setup on your cluster and **watch rolling updates in action** using:

```bash
kubectl get pods -w
```
