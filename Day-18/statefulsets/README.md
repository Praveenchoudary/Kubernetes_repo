

## 🔹 What is a StatefulSet?

A **StatefulSet** is a Kubernetes workload API object used for managing **stateful applications**.
Unlike a Deployment, it provides:

* ✅ Stable, unique pod names (`app-0`, `app-1`, …)
* ✅ Persistent storage for each pod (via PVCs)
* ✅ Ordered deployment, scaling, and termination
* ✅ Stable network identities via **Headless Services**

👉 Use StatefulSets for **databases, messaging systems, and search engines** that need **consistent identity and storage**.

---

## 🔹 Key Features

1. **Stable Pod Identity** → Pods are not interchangeable (`mysql-0`, `mysql-1`).
2. **Stable Storage** → Each pod has its own PVC.
3. **Ordered Startup/Shutdown** → Pods start & stop one by one in sequence.
4. **Headless Service** → Enables direct pod-to-pod communication.

---

## 🔹 Real-Time Use Cases

### 1️⃣ MySQL Cluster (Database)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  ports:
    - port: 3306
  clusterIP: None
  selector:
    app: mysql
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: password123
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 2Gi
```

📌 Pods created: `mysql-0`, `mysql-1`, `mysql-2`
📌 Each pod gets its own PVC: `mysql-data-mysql-0`, etc.

---

### 2️⃣ Kafka Cluster (Messaging System)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kafka
spec:
  clusterIP: None
  ports:
    - port: 9092
  selector:
    app: kafka
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: kafka
spec:
  serviceName: kafka
  replicas: 3
  selector:
    matchLabels:
      app: kafka
  template:
    metadata:
      labels:
        app: kafka
    spec:
      containers:
      - name: kafka
        image: bitnami/kafka:latest
        env:
        - name: KAFKA_ZOOKEEPER_CONNECT
          value: zookeeper:2181
        - name: KAFKA_BROKER_ID
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        volumeMounts:
        - name: kafka-data
          mountPath: /bitnami/kafka
  volumeClaimTemplates:
  - metadata:
      name: kafka-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 5Gi
```

📌 Stable brokers: `kafka-0`, `kafka-1`, `kafka-2`
📌 Each broker keeps its data partitions even after restart.

---

### 3️⃣ Elasticsearch Cluster (Search & Analytics)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: elasticsearch
spec:
  clusterIP: None
  ports:
    - port: 9200
    - port: 9300
  selector:
    app: elasticsearch
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: elasticsearch
spec:
  serviceName: elasticsearch
  replicas: 3
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
    spec:
      containers:
      - name: elasticsearch
        image: docker.elastic.co/elasticsearch/elasticsearch:8.14.1
        env:
        - name: discovery.seed_hosts
          value: "elasticsearch-0.elasticsearch,elasticsearch-1.elasticsearch,elasticsearch-2.elasticsearch"
        - name: cluster.initial_master_nodes
          value: "elasticsearch-0,elasticsearch-1,elasticsearch-2"
        - name: node.name
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        volumeMounts:
        - name: data
          mountPath: /usr/share/elasticsearch/data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 5Gi
```

📌 Pods: `elasticsearch-0`, `elasticsearch-1`, `elasticsearch-2`
📌 Each node retains its index data across restarts.

---

## 🔹 When to Use StatefulSet vs Deployment?

| **Feature**  | **Deployment**                | **StatefulSet**                |
| ------------ | ----------------------------- | ------------------------------ |
| Pod identity | Random                        | Stable (`pod-0`, `pod-1`)      |
| Storage      | Shared/ephemeral              | Dedicated PVC per pod          |
| Use case     | Stateless apps (Nginx, React) | Stateful apps (DBs, Kafka, ES) |
| Scaling      | Fast                          | Ordered & consistent           |

---

## 🔹 Real-Life Analogy

* **Deployment = Dormitory** 🏠 (any bed works, no fixed identity)
* **StatefulSet = Hotel Rooms** 🏨 (Room 101 is always Room 101, with guest belongings inside)

---

## 🚀 How to Run

```bash
kubectl apply -f mysql-statefulset.yaml
kubectl apply -f kafka-statefulset.yaml
kubectl apply -f elasticsearch-statefulset.yaml
```

Check pods:

```bash
kubectl get pods -o wide
```

Check PVCs:

```bash
kubectl get pvc
```

---

## 📌 Conclusion

* Use **StatefulSet** when your application needs **stable identities and persistent storage**.
* Great for **databases, message queues, and distributed systems**.
* Not suitable for stateless frontend apps (use **Deployment** instead).

---
