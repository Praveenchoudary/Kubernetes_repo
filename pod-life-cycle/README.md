  <img width="633" height="425" alt="image" src="https://github.com/user-attachments/assets/2707f7c6-a91c-4740-a914-ccd6cac64a5a" />

---

# 🐳 Kubernetes Pod Lifecycle Examples

This repository demonstrates **Kubernetes Pod Lifecycle** with practical examples and YAML files. Learn about Pod phases, restart policies, probes, graceful termination, and CrashLoopBackOff.

---

## 📌 Pod Lifecycle Overview

Pods are the smallest deployable units in Kubernetes. Each Pod goes through several **phases** during its lifecycle:

| Phase           | Description                                                |
| --------------- | ---------------------------------------------------------- |
| 🟡 **Pending**  | Pod accepted but containers not ready.                     |
| 🟢 **Running**  | Pod bound to a node and at least one container is running. |
| ✅ **Succeeded** | All containers completed successfully.                     |
| ❌ **Failed**    | At least one container terminated in failure.              |
| ❓ **Unknown**   | State of Pod could not be determined.                      |

> Pods are ephemeral. If a Pod is deleted or a node fails, Kubernetes may replace it with a new Pod (different UID).

---


### 1️⃣ Pending Pod

Simulates a Pod stuck in **Pending** due to a non-existent image.
**File:** `pod-pending.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pending-pod
spec:
  containers:
  - name: invalid-container
    image: nonexistingimage:latest
```

---

### 2️⃣ Running Pod

A Pod running **Nginx**.
**File:** `pod-running.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: running-pod
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
    ports:
    - containerPort: 80
```

---

### 3️⃣ Restart Policy: OnFailure

Pod fails and restarts automatically based on `OnFailure`.
**File:** `restart-policy-onfailure.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: restart-onfailure-pod
spec:
  restartPolicy: OnFailure
  containers:
  - name: fail-container
    image: busybox
    command: ["sh", "-c", "exit 1"]
```

---

### 4️⃣ Probes Pod

Demonstrates **liveness, readiness, and startup probes**.
**File:** `probe-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: probe-pod
spec:
  containers:
  - name: app-container
    image: nginx
    ports:
    - containerPort: 80
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 2
      periodSeconds: 5
    startupProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
```

---

### 5️⃣ Graceful Termination

Pod with **preStop hook** and **termination grace period**.
**File:** `terminate-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: terminate-pod
spec:
  terminationGracePeriodSeconds: 20
  containers:
  - name: app-container
    image: nginx
    lifecycle:
      preStop:
        exec:
          command: ["sh", "-c", "echo 'Pod is terminating...' && sleep 10"]
```

---

### 6️⃣ CrashLoopBackOff Pod

Pod fails repeatedly, triggering **CrashLoopBackOff**.
**File:** `crashloopbackoff-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: crashloop-pod
spec:
  restartPolicy: Always
  containers:
  - name: crash-container
    image: busybox
    command: ["sh", "-c", "echo 'Starting container'; exit 1"]
```

---

## ⚡ Useful Commands

```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl delete pod <pod-name> --grace-period=30
```

---

* Containers in **CrashLoopBackOff** → fail and restart repeatedly.
* **PreStop hooks** ensure cleanup before termination.
* **Readiness probes** prevent traffic to unready Pods.
* **Startup probes** help long-starting containers avoid premature restarts.
* **RestartPolicy** defines whether Pods restart automatically.

---


If you want, I can now **package all YAML files in a ready-to-clone GitHub repo structure** with this README so it’s completely ready for push.

Do you want me to do that?
