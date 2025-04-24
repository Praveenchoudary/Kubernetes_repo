# 🚀 Kubernetes Labels and Selectors

## 🏷️ Labels

Labels are key-value pairs that are attached to Kubernetes objects such as Pods, Nodes, etc. They are used to organize and select subsets of objects. Labels are useful for grouping, filtering, and identifying resources in a Kubernetes cluster.

### 🔹 Key Features:
- Labels are defined as key-value pairs.
- Multiple labels can be assigned to a single Kubernetes object.
- They help in filtering resources efficiently—similar to **tags in AWS**.
- Commonly used for distinguishing different app versions, environments, or teams.

### 🧪 Example: Creating Pods with Labels

#### ✅ Pod related to Swiggy
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    app: swiggy
spec:
  containers:
    - name: cont1
      image: nginx
      ports:
        - containerPort: 80

✅ Pod related to Zomato

apiVersion: v1
kind: Pod
metadata:
  name: pod2
  labels:
    app: zomato
spec:
  containers:
    - name: cont1
      image: nginx
      ports:
        - containerPort: 80

🔍 Useful Commands

    View Labels on Pods

kubectl get pods --show-labels

Add Label to an Existing Pod

kubectl label pod <pod-name> <label-key>=<label-value>

Filter Pods by Label

kubectl get pods -l app=swiggy

Exclude Pods by Label

    kubectl get pods -l app!=swiggy

🎯 Selectors

Selectors allow users to filter Kubernetes resources based on labels. They help in defining relationships between resources and are frequently used in services, deployments, and replica sets.
🔸 Types of Selectors
✅ Equality-based Selectors

    Use = or !=

    Match resources with a specific label value

kubectl get pods -l app=swiggy
kubectl get pods -l app!=zomato

✅ Set-based Selectors

    Use in, notin, or exists operators

    Match multiple label values

kubectl get pods -l 'app in (swiggy,zomato)'
kubectl get pods -l 'app notin (swiggy)'
kubectl get pods -l app

📚 Summary
Feature	Description
Labels	Used to tag and organize Kubernetes objects
Selectors	Used to filter resources based on labels
Types of Selectors	Equality-based (=, !=) and Set-based (in, notin)
