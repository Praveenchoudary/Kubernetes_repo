## 🔹 What is a DaemonSet?

A **DaemonSet** in Kubernetes ensures that a **copy of a Pod runs on every Node** (or specific Nodes if you use selectors).

* When new nodes are added to the cluster, Pods are automatically added to them.
* When nodes are removed, Pods are garbage-collected.
* Deleting the DaemonSet will clean up the Pods it created.

👉 It’s like a **background service** that should run **everywhere** in the cluster.

---

## 🔹 Real-life Analogy

Think of **DaemonSet** like **electricity meters** installed in every house of a city:

* Every house (Kubernetes Node) must have a meter (Pod).
* If a new house is built, the electricity company automatically installs a meter (DaemonSet schedules a Pod).
* If a house is demolished, the meter is removed (Pod is deleted).

So, **DaemonSet = ensures one Pod per Node, just like electricity meter per house.**

---

## 🔹 Real-world Use Cases of DaemonSets

1. **Log Collection Agents** – e.g., Fluentd, Filebeat → Run on every node to collect logs.
2. **Monitoring Agents** – e.g., Prometheus Node Exporter → Run on every node to expose system metrics.
3. **Networking Plugins** – e.g., Calico, Weave → Each node needs networking Pods.
4. **Security Agents** – e.g., Falco → Monitor all nodes for threats.

---

## 🔹 Example: Node Exporter DaemonSet

Here’s a **YAML manifest** for a Prometheus Node Exporter DaemonSet:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      containers:
      - name: node-exporter
        image: prom/node-exporter:latest
        ports:
        - containerPort: 9100
          name: metrics
      tolerations:
      - effect: NoSchedule
        operator: Exists
      hostNetwork: true
      hostPID: true
```

✅ This will run **Node Exporter on every Node** → collects system metrics like CPU, memory, disk usage.

---

## 🔹 GitHub README.md (ready to use)

Here’s a GitHub-style README you can directly put in a repo:

````markdown
# 🚀 Kubernetes DaemonSet Example

## 📌 What is a DaemonSet?
A **DaemonSet** ensures that a specific Pod runs on **every Node** in a Kubernetes cluster.  
If new nodes are added, the Pod will be automatically scheduled on them.  
If nodes are removed, the Pods are deleted.

---

## 🏠 Real-life Analogy
Think of a DaemonSet like **electricity meters** in a city:
- Every house (Node) must have one meter (Pod).
- New house? → Meter is installed automatically.
- House demolished? → Meter is removed.

👉 DaemonSet guarantees **one Pod per Node**.

---

## 🔧 Common Use Cases
- 📊 **Monitoring agents** (Prometheus Node Exporter, Datadog agent)
- 📜 **Log collectors** (Fluentd, Filebeat)
- 🌐 **Networking plugins** (Calico, Weave)
- 🔐 **Security agents** (Falco, Sysdig)

---

## 📂 Example: Node Exporter DaemonSet

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      containers:
      - name: node-exporter
        image: prom/node-exporter:latest
        ports:
        - containerPort: 9100
          name: metrics
      tolerations:
      - effect: NoSchedule
        operator: Exists
      hostNetwork: true
      hostPID: true
````

---

## 🚀 How to Apply

```bash
kubectl apply -f daemonset.yaml
kubectl get pods -n monitoring -o wide
```

You’ll see **one Pod per Node** in your cluster.

---

## ✅ Output Example

```bash
kubectl get pods -n monitoring -o wide

NAME                 READY   STATUS    NODE
node-exporter-abcd   1/1     Running   worker-node-1
node-exporter-efgh   1/1     Running   worker-node-2
node-exporter-ijkl   1/1     Running   worker-node-3
```

---

## 🎯 Summary

* DaemonSet = Run Pods on **every Node**
* Perfect for **monitoring, logging, networking, security agents**
* Automatically adjusts when nodes are added or removed

---

```
