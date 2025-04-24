# 📦 Kubernetes Deployment

This section contains an example of a **Kubernetes Deployment**, a powerful object used to manage the lifecycle of your application containers on a Kubernetes cluster.

---

![image](https://github.com/user-attachments/assets/4c8eb40b-2aea-4488-b22b-0d7502b2d1d6)


## 🧭 Introduction

A **Deployment** in Kubernetes provides a declarative way to describe the desired state of your application. It ensures the correct number of **Pods** are running, allows for **rolling updates**, and handles **automatic recovery** of failed Pods. This makes it ideal for managing scalable and resilient applications in production.

---

# 📦 Kubernetes Deployment

This document explains how to use the **Kubernetes Deployment** resource to manage application workloads with rolling updates, auto-scaling, and self-healing capabilities.

---

## 🧭 Introduction

A **Deployment** ensures that a specific number of Pods are always running and updated. It automatically replaces failed Pods, supports rolling updates, and helps manage changes to container images or configuration over time — all with minimal effort.

---

## 🌟 Key Features & Examples

### 1. ✅ Declarative Configuration

Write your deployment in a YAML file and apply it to your cluster.

```bash
kubectl apply -f deployment.yaml

2. 🔁 Rolling Updates

Update container images with zero downtime.

kubectl set image deployment/my-app-deployment my-app-container=nginx:1.21

3. 🔄 Rollback Support

Revert to the previous version if something goes wrong.

kubectl rollout undo deployment/my-app-deployment

4. ♻️ Self-Healing

Kubernetes will restart Pods if they crash — no action needed!

To simulate:

kubectl delete pod <pod-name>
# Kubernetes will create a new one automatically

5. 📈 Horizontal Scaling

Easily scale the number of Pods running your app.

kubectl scale deployment my-app-deployment --replicas=5

6. 🔍 Rollout Status Tracking

Watch the progress of an ongoing update.

kubectl rollout status deployment/my-app-deployment

---

