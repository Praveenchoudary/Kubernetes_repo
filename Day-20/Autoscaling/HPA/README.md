:
---

# 🚀 Kubernetes Horizontal Pod Autoscaler (HPA)

This guide explains how to implement **Horizontal Pod Autoscaling (HPA)** in Kubernetes using **Metrics Server**, a sample **Nginx deployment**, and an **HPA configuration**.

---

<img width="640" height="444" alt="image" src="https://github.com/user-attachments/assets/f88196d0-3dae-4af2-aa2e-e77af925af99" />


## 📌 Prerequisites

* ✅ A running Kubernetes cluster
* ✅ `kubectl` configured to access the cluster
* ✅ Metrics Server installed

---


## 🏢 Real-Life Example

Think of an **E-commerce website** 🛒:

* During **Diwali Sale** or **Black Friday**, thousands of users visit at once.
* Without autoscaling, your website might **crash** 😱.
* With **HPA**, Kubernetes **adds more pods automatically** when CPU usage increases, and reduces them when traffic goes down.


## 🛠️ Steps to Implement HPA

### **Step 1: Install Metrics Server**

HPA requires metrics to make scaling decisions. Install the **Metrics Server**:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

---

### **Step 2: Deploy Application**

Create a deployment file `deploy.yml` with your application definition (including resource requests & limits).

Apply the deployment:

```bash
kubectl apply -f deploy.yml
```

---

### **Step 3: Create HPA**

Create an HPA definition file `hpa.yml` that targets your deployment.

Apply the HPA:

```bash
kubectl apply -f hpa.yml
```

---

### **Step 4: Verify HPA**

Check if HPA is created:

```bash
kubectl get hpa
```

Watch pod scaling in real time:

```bash
kubectl get pods -w
```

---

### **Step 5: Generate Load**

1. Get inside a running pod:

   ```bash
   kubectl exec -it <pod-name> -- bash
   ```

2. Install stress tool:

   ```bash
   apt update -y
   apt install stress -y
   ```

3. Run CPU stress:

   ```bash
   stress --cpu 2 --timeout 300
   ```

⚡ This will increase CPU utilization.
📈 When CPU usage > **50%**, HPA will **create new pods (up to 5)**.
🧹 When usage decreases, pods scale down (minimum **1 pod**).

---

## 🎯 Benefits of HPA

* ⚡ **Efficient Resource Utilization**
* 🚀 **Improved Application Performance**
* 💰 **Cost Savings**

---

## 🖼️ Commands Summary

```bash
# Install Metrics Server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Deploy App
kubectl apply -f deploy.yml

# Apply HPA
kubectl apply -f hpa.yml

# Check HPA
kubectl get hpa

# Watch Pods Scale
kubectl get pods -w
```






