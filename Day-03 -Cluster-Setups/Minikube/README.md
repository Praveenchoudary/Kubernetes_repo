# ☸️ Minikube Kubernetes Cluster Setup

Minikube is a lightweight Kubernetes implementation that creates a virtual cluster on your local machine for learning and development.

---

## 📋 Prerequisites

- **Operating System:** Windows, Linux, or macOS
- **Virtualization:** Docker, VirtualBox, KVM, Hyper-V, etc.
- **Tools Installed:**
  - [Kubectl](https://kubernetes.io/docs/tasks/tools/)
  - [Minikube](https://minikube.sigs.k8s.io/docs/start/)

---

## 🛠️ Install `kubectl`

```bash
sudo apt update
sudo apt install -y curl
curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client

## 🛠️ Install `minikube`

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
minikube version

## 🛠️ Start minikube`
minikube start --driver=docker
