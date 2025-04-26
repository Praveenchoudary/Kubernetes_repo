# 📂 Kubernetes Namespaces 

---

## 📘 What Are Namespaces?

Namespaces are **virtual clusters** inside a Kubernetes cluster. They are used to **logically isolate and organize resources** such as Deployments, Services, ConfigMaps, etc.

- Names of resources must be unique **within a namespace**, but can be reused **across namespaces**.
- Namespace-based scoping is only applicable for namespaced objects.
- **Cluster-wide objects** like `StorageClass`, `Nodes`, `PersistentVolumes` are **not namespaced**.
  
![image](https://github.com/user-attachments/assets/d0d392c0-ec4e-4457-96b9-681147cdd226)

---

## ✅ Why Use Namespaces?

Namespaces are essential for:

- Organizing a large number of Kubernetes resources.
- Creating **virtual clusters** for different **teams** or **projects**.
- Avoiding conflicts between different applications or environments.
- Applying **RBAC**, **network policies**, and **resource quotas** on a per-namespace basis.

  ![image](https://github.com/user-attachments/assets/5e0f32b1-5d6f-4a78-a718-81accfb93901)


---

## 💡 Real-world Scenarios

### 🔸 Team-based Isolation

![image](https://github.com/user-attachments/assets/eec1462c-6780-40f5-b04b-a2d82b13109a)

Assigning each team its own namespace:

- `team-a` → managed by Team A
- `team-b` → managed by Team B

> Teams cannot access or modify each other’s resources.  
> Cluster admins can set access limits using RBAC.

---

### 🔸 Environment Separation

![image](https://github.com/user-attachments/assets/f11b672c-8a29-4abd-9295-2ae0f66fa781)


Separate namespaces for different environments:

- `dev` → for development
- `staging` → for pre-production
- `prod` → for production

> Dev testing won’t affect stable production workloads.

---

### 🔸 Logical Grouping

![Uploading image.png…]()


Group components based on their roles:

- `database` → MySQL, Redis, RabbitMQ, etc.
- `monitoring` → Prometheus, Grafana, etc.
- `application-services` → Main application deployments and services.

---

## 📦 Default Namespaces in Kubernetes

When a Kubernetes cluster is provisioned, the following namespaces are created by default:

| Namespace         | Description |
|------------------|-------------|
| `default`         | Used when no other namespace is specified. |
| `kube-system`     | Holds system-critical pods like `kube-dns`, `kube-proxy`, etc. Do not use for personal resources. |
| `kube-public`     | Contains publicly readable data (readable without authentication). Mostly for cluster info. |
| `kube-node-lease` | Stores Lease objects used for node heartbeat checks by kubelet. Ensures node health detection. |

---

## 🚀 How to Work with Namespaces

### Create a Namespace

```bash
kubectl create namespace dev
