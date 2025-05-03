# 📦 Kubernetes Volumes

This section covers how volumes work in Kubernetes, with a focus on **PersistentVolumes (PV)**, **PersistentVolumeClaims (PVC)**, and the difference between **Static** and **Dynamic Provisioning**.

---

## 📘 What is a Volume?

A **Volume** in Kubernetes is a storage directory accessible to containers in a Pod. Unlike container-local storage (which is lost when a container restarts), a volume’s data can persist across restarts or even be shared across containers.

---

## 📂 Volume Types in Kubernetes

### 1. **EmptyDir**
- Temporary directory that exists as long as the Pod is running.
- Useful for scratch space.

### 2. **hostPath**
- Mounts a file or directory from the host node's filesystem.
- Not recommended for production due to tight coupling with node.

### 3. **PersistentVolume (PV) / PersistentVolumeClaim (PVC)**
- Abstracts the underlying storage system (NFS, AWS EBS, GCE PD, etc.).
- Supports **static** and **dynamic provisioning**.

---

## 🆚 Static vs Dynamic Provisioning

| Feature               | Static Provisioning              | Dynamic Provisioning                     |
|----------------------|----------------------------------|------------------------------------------|
| PV creation          | Manually created by admin        | Automatically created by Kubernetes      |
| PVC binding          | Matches existing PVs             | Triggers automatic PV creation           |
| Flexibility          | Less                             | High                                     |
| Recommended for      | Custom setups                    | Cloud-native and dynamic environments    |

---
