

````markdown
# 🚀 Migrating from Docker (dockershim) to containerd in Kubernetes

Kubernetes v1.24+ **removes support for Docker Engine** via `dockershim`. This means Kubernetes now interacts **directly** with modern container runtimes like **containerd** using the **Container Runtime Interface (CRI)**.

---

## 📜 Background

### ⛔ Before Kubernetes v1.24:
- Kubernetes did **not** talk directly to Docker Engine.
- It used a shim layer called `dockershim` to **bridge communication** between the **kubelet** and **Docker**.
- Docker itself used `containerd` internally, but Kubernetes couldn’t talk to it directly.

### ✅ After Kubernetes v1.24:
- `dockershim` is **removed**.
- Kubernetes **natively supports** container runtimes like `containerd` and `CRI-O` via **CRI**.
- This results in **simpler, faster, and more maintainable** architecture.

---

## 🔧 Example Scenarios

### 📦 Docker (with dockershim) — *Kubernetes v1.20*

1. You deploy a pod using `kubectl apply`.
2. Kubernetes sends the request to the `kubelet`.
3. `kubelet` uses `dockershim` → Docker Engine → `containerd` (internally).
4. Container is pulled and started via Docker.

```text
kubelet <---> dockershim <---> Docker Engine <---> containerd
````

---

### ⚙️ containerd (native) — *Kubernetes v1.27*

1. You deploy a pod.
2. `kubelet` talks **directly** to `containerd` via CRI.
3. containerd pulls images, manages containers, networking, storage.

```text
kubelet <---> containerd
```

![Uploading image.png…]()


* ✅ No Docker.
* ✅ No dockershim.
* ✅ Leaner, cleaner, modern setup.

---

## 🔍 Comparison Table

| 🧩 Aspect            | 🐳 Docker Engine (with dockershim)                | 📦 containerd (native CRI)    |
| -------------------- | ------------------------------------------------- | ----------------------------- |
| Kubernetes Version   | v1.23 and below                                   | v1.24 and above               |
| Container Runtime    | Docker Engine (via dockershim)                    | containerd (direct)           |
| Communication Flow   | kubelet → dockershim → Docker Engine → containerd | kubelet → containerd          |
| Runtime Installation | Install full Docker Engine                        | Install containerd standalone |
| Performance          | Extra layer adds overhead                         | Lightweight and efficient     |
| CRI Support          | Deprecated and removed in v1.24                   | Fully CRI-compliant           |
| Maintenance          | More components to troubleshoot                   | Fewer moving parts            |

---

## 🔄 What Should You Do?

✅ If you're using **Kubernetes v1.24 or newer**, switch to `containerd`.

🔧 You can install containerd using your OS package manager (e.g., `apt`, `yum`) or use containerd as part of your cloud distribution (EKS, GKE, AKS).

📘 [Kubernetes: Remove Docker as a Container Runtime](https://kubernetes.io/blog/2020/12/02/dont-panic-kubernetes-and-docker/)



