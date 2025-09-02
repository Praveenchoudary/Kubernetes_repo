
---

# 🚀 Kubernetes Horizontal Pod Autoscaler (HPA) with NGINX

This project demonstrates how to implement **Horizontal Pod Autoscaling (HPA)** in Kubernetes using an **NGINX deployment**.

We’ll also generate **stress (CPU load)** to see how the HPA automatically scales pods up and down.

---

## 🏢 Real-Life Example

Think of an **E-commerce website** 🛒:

* During **Diwali Sale** or **Black Friday**, thousands of users visit at once.
* Without autoscaling, your website might **crash** 😱.
* With **HPA**, Kubernetes **adds more pods automatically** when CPU usage increases, and reduces them when traffic goes down.

This ensures:
✅ High availability
✅ Cost efficiency
✅ No manual intervention

---

## 🛠️ Steps to Implement

### 1️⃣ Prerequisites

* Kubernetes cluster running (Minikube, EKS, GKE, AKS, etc.)
* `kubectl` installed
* Metrics server installed (needed for HPA)

👉 Install metrics server (if not already):

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

---

### 2️⃣ Create Deployment & Service (NGINX)

* Create a `deployment.yaml` for NGINX (3 replicas to start).
* Create a `service.yaml` (ClusterIP or NodePort).

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

---

### 3️⃣ Create HPA

* Define an `hpa.yaml` that scales NGINX between **2–10 replicas** based on **CPU usage (50%)**.

```bash
kubectl apply -f hpa.yaml
```

Check HPA:

```bash
kubectl get hpa
```

---

### 4️⃣ Generate Stress (CPU Load)

We’ll use a stress container to simulate high traffic 📈.

```bash
kubectl run -i --tty load-generator --rm --image=busybox -- sh
```

Inside container:

```bash
while true; do wget -q -O- http://<nginx-service-ip>; done
```

This creates continuous requests → increases CPU usage → HPA scales pods 🚀.

---

### 5️⃣ Monitor Scaling

Check NGINX pod scaling in real time:

```bash
kubectl get pods -w
```

You’ll see pods **increasing** when load is high, and **decreasing** when load drops.

---

## 📊 Verification

* Run:

```bash
kubectl describe hpa
```

* You’ll see metrics like CPU utilization and replica count.

---

## 🎯 Key Learnings

1. HPA prevents app crashes during sudden traffic spikes.
2. Saves cost by scaling **down** when load is low.
3. Works with **CPU, Memory, and custom metrics**.


Would you like me to also **write the folder structure suggestion** (like `/manifests/deployment.yaml`, `/manifests/service.yaml`, `/manifests/hpa.yaml`) for your GitHub repo 📂 so it looks production-ready?
