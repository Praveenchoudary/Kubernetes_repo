# 🚀 Kubernetes Pod Scheduling Strategies

This repository contains comprehensive explanations and examples for various Kubernetes **pod scheduling techniques**, including:

- ✅ Node Selector
- 🎯 Node Affinity
- 🔄 Pod Anti-Affinity
- ⚠️ Taints and Tolerations

These mechanisms help control **where** pods are scheduled in a Kubernetes cluster.

---

## 📌 1. Node Selector

### 🔍 What is it?

`nodeSelector` is the simplest way to constrain a Pod to run only on nodes with a specific label.

### ✅ Use Case

You want your Pod to run only on nodes with SSDs.

### 🛠️ How to Use

1. **Label your node:**

kubectl label nodes <node-name> disktype=ssd

# Kubernetes Node Affinity and Node Selector 🌍

In Kubernetes, **Node Affinity** and **Node Selector** are mechanisms that allow you to control where your pods are scheduled within the cluster. They both influence the Kubernetes scheduler by setting rules based on node labels. 📝

### Node Selector 🧑‍💻

Node Selector is the simplest form of node scheduling. You can define key-value pairs for nodes and use these pairs to control where the pods are placed. However, Node Selector is strict, meaning that a pod will only be scheduled on a node that matches the specified selector. 🚫

#### Example of Node Selector:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      nodeSelector:
        node-arch: windows
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

In this example, the pod will only be scheduled on nodes with the label node-arch: windows. If no matching node is available, the pod will remain in a Pending state. ⏳
Node Affinity 📍

Node Affinity is a more flexible and expressive way to influence the placement of pods. It allows you to specify preferred and required node affinity rules.

    Preferred Node Affinity: The pod prefers certain nodes, but if no matching nodes are found, it will be scheduled on any available node. 😎

    Required Node Affinity: The pod must be scheduled on a node that matches all of the specified labels. If no matching node is found, the pod will remain in a Pending state. 🚨

Preferred Node Affinity 💡:

This option is useful when you have a preference for certain nodes, but it's not a strict requirement.

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      affinity:
        nodeAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            preference:
              matchExpressions:
              - key: node-arch
                operator: In
                values:
                - windows
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

In this example, the pod prefers nodes with the label node-arch: windows, but if no such nodes are available, the pod can still be scheduled on other nodes. 🌐
Required Node Affinity 🔒:

This option enforces that the pod will only be scheduled on nodes that match the specified labels. If no matching nodes are found, the pod will remain in a Pending state. 🚧

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - nodeSelectorTerms:
            - matchExpressions:
              - key: app
                operator: In
                values:
                - myapp
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

In this example, the pod will only be scheduled on nodes with the label app: myapp. If no such nodes are available, the pod will remain in a Pending state. ⏳
Differences Between Node Selector and Node Affinity ⚖️

    Node Selector: The pod is forced to schedule on a node with a matching label. If no matching node is found, the pod remains in a Pending state. 🔒

    Node Affinity (Preferred): The pod prefers nodes with specific labels. If no matching nodes are found, the pod can still be scheduled on any available node in the cluster. 🌍

    Node Affinity (Required): The pod will only be scheduled on nodes that match the specified labels. If no matching nodes are found, the pod will remain in a Pending state. 🚫

Conclusion 🎯

    Use Node Selector for simple, strict node scheduling requirements. ❗

    Use Node Affinity for more flexible and expressive control over pod placement with the ability to specify both preferred and required rules. 🔄
