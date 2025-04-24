# ☸️ Day 02 – Kubernetes Architecture

---

## 🔧 What is Kubernetes Architecture?

Kubernetes uses a **master-worker architecture** also referred to as the **Control Plane and Data Plane (Nodes)**. The Control Plane makes decisions about the cluster, while the Worker Nodes (Data Plane) run applications and containers.

---

## 🖼️ Kubernetes Architecture Diagram

![image](https://github.com/user-attachments/assets/a5f02867-3ef4-4a20-8e98-62fb2ceff056)

---

## 🧠 Control Plane Components (Master Node)

The **Control Plane** manages the cluster and ensures the desired state is maintained.

### 1. **API Server**
- The main entry point to the cluster.
- Exposes the Kubernetes API.
- Receives commands from `kubectl`, UI, or automation tools.
- Validates and processes REST requests.

### 2. **Scheduler**
- Assigns newly created pods to suitable nodes.
- Decisions are based on resource availability, affinity, taints, and tolerations.
- Talks to the API server to receive pod definitions.

### 3. **etcd**
- A distributed key-value store.
- Stores all cluster data, including pod definitions, secrets, config maps, and more.
- Acts as the source of truth for the cluster.

### 4. **Controller Manager**
- Runs built-in controllers (like Deployment, ReplicaSet, StatefulSet).
- Ensures the current state matches the desired state.
- Responsible for operations like scaling, rolling updates, etc.

### 5. **Cloud Controller Manager**
- Integrates Kubernetes with cloud providers (AWS, GCP, Azure).
- Manages services like load balancers, storage volumes, and node lifecycle in the cloud environment.

---

## ⚙️ Data Plane Components (Worker Node)

Worker nodes are responsible for running the actual application workloads.

### 1. **Kubelet**
- Agent on each node that ensures containers are running in pods.
- Communicates with the API Server.
- Reports the health and status of the node/pods.

### 2. **Kube-Proxy**
- Handles network routing within the cluster.
- Assigns IP addresses to pods and services.
- Provides basic load balancing across services.

### 3. **Container Runtime**
- Software responsible for running containers (e.g., Docker, containerd, CRI-O).
- Interfaces with the OS to run and manage containers.

### 4. **Pod**
- The smallest and simplest unit in the Kubernetes object model.
- A pod contains one or more containers with shared networking and storage.
- Managed by higher-level objects like Deployments or ReplicaSets.

---

## ✅ Summary Table

| Layer           | Component             | Description                                                                 |
|----------------|------------------------|-----------------------------------------------------------------------------|
| Control Plane   | API Server             | Entry point, exposes Kubernetes API                                        |
|                 | Scheduler              | Schedules pods to nodes                                                    |
|                 | etcd                   | Cluster data storage (key-value)                                           |
|                 | Controller Manager     | Maintains desired cluster state                                            |
|                 | Cloud Controller Mgr   | Integrates with cloud provider features                                    |
| Worker Node     | Kubelet                | Ensures containers are running, communicates with API server               |
|                 | Kube-Proxy             | Manages pod networking and routing                                         |
|                 | Container Runtime      | Runs containers using Docker/containerd/CRI-O                              |
|                 | Pod                    | Smallest unit in Kubernetes, runs application containers                   |

---
