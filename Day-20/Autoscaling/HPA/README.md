
---

# 🚀 Kubernetes Horizontal Pod Autoscaler (HPA)

This project demonstrates how to use **Horizontal Pod Autoscaler (HPA)** in Kubernetes to **scale an NGINX deployment automatically** based on CPU usage.

HPA ensures your application can handle **high traffic loads dynamically** without manual intervention 💡.

---

## 📖 Real-Life Example

Imagine you run an **e-commerce website** 🛒:

* During **normal hours**, you may only need **2-3 NGINX pods** to handle requests.
* But during a **flash sale** ⚡ or **festival season** 🎉, traffic suddenly spikes 🚦.
* Without autoscaling, your app could **crash** 😢 due to overload.
* With **HPA**, Kubernetes will automatically **increase pods** (e.g., from 2 ➝ 10) to handle traffic.
* When traffic reduces, it will **scale down** again, saving 💰 on resources.

---

## 🛠️ Prerequisites

Make sure you have:

* ✅ Kubernetes cluster (minikube, kind, EKS, GKE, AKS, etc.)
* ✅ `kubectl` installed & configured
* ✅ Metrics server installed (HPA needs metrics)

👉 Install metrics server if not already:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```


## 📜 Deployment YAML

### 🔹 NGINX Deployment & Service (`nginx-deployment.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
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
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "100m"
          limits:
            cpu: "200m"
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

---

### 🔹 HPA YAML (`hpa.yaml`)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

⚡ This means:

* Minimum **2 pods**
* Maximum **10 pods**
* Scale up when CPU usage > **50%**

---

## 🚀 Steps to Deploy

1️⃣ Apply the NGINX deployment & service:

```bash
kubectl apply -f nginx-deployment.yaml
```

2️⃣ Apply the HPA:

```bash
kubectl apply -f hpa.yaml
```

3️⃣ Check HPA status:

```bash
kubectl get hpa
```

---

## 🔥 Stress Testing the Autoscaler

We will use **Apache Benchmark (`ab`)** to simulate traffic:

```bash
kubectl run -i --tty load-generator --image=busybox /bin/sh
```

Inside the pod, run:

```bash
while true; do wget -q -O- http://nginx-service; done
```

This creates continuous load → CPU usage increases → HPA scales pods 🚀

---

## 📊 Verify Scaling

Check pods scaling in action:

```bash
kubectl get pods -w
```

You will see pods scaling up when traffic increases ⚡ and scaling down when load reduces ⬇️.

---

## 🎯 Outcome

✅ Automatic pod scaling
✅ High availability during traffic spikes
✅ Cost efficiency by scaling down when not needed

---


