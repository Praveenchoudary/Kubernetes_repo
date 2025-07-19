
---

## 🚦 Kubernetes Pod Disruption Budget (PDB)

---

### 📘 What is a Pod Disruption Budget?

A **Pod Disruption Budget (PDB)** in Kubernetes helps you **protect your application availability** when the cluster experiences **voluntary disruptions** like:

* 🧹 Node drains (e.g., for maintenance)
* 🔄 Node upgrades
* 📏 Cluster scaling events

It sets a **limit on how many pods can be unavailable** at a time so your service stays online 💡

---

### 🧠 Why PDB Matters

Imagine you have a **web application** with multiple replicas running. If all the pods go down together (even for a brief moment), users will see errors 😞.

With a PDB:
✅ You ensure that **some pods always stay up**,
❌ Even if someone tries to drain a node or do rolling maintenance.

---

### ⚙️ Types of PDB Rules

You define **either** of the following:

1. ✅ `minAvailable`:

   > The **minimum number of pods** that should be available at any time.
   > Example: `"minAvailable: 2"` ensures at least 2 pods are running.

2. 🚫 `maxUnavailable`:

   > The **maximum number of pods** that can be unavailable.
   > Example: `"maxUnavailable: 1"` means never more than 1 pod down.

You **can’t use both** in a single PDB.

---

### 🔁 Voluntary vs Involuntary Disruptions

| Type          | Examples                                       | PDB Applies? |
| ------------- | ---------------------------------------------- | ------------ |
| ✅ Voluntary   | `kubectl drain`, node upgrades, scaling events | ✔️ Yes       |
| ❌ Involuntary | Node crash, hardware failure, OOM kills        | ❌ No         |

PDB **only protects against voluntary disruptions**.

---

### 🔍 Real-world Scenario

Suppose:

* You run a **Deployment** with 3 pods.
* You define a PDB with `minAvailable: 2`.

Now:

* If you try to **drain a node**, only **1 pod can be evicted**.
* If draining would bring availability below 2, **Kubernetes blocks it** ❌.

This ensures **your app stays online** even during changes.

---

### 🔧 How Does It Work?

When an action tries to evict a pod (like a drain), Kubernetes checks:

* 👉 Does this action **violate the PDB**?
* ✅ If **not**, the eviction is allowed.
* ❌ If **yes**, it will **fail the operation** (e.g., drain is halted).

---

### 📊 Check PDB Status

Use this command to see how many pods can be disrupted:

```bash
kubectl get pdb
```

Look for the `ALLOWED DISRUPTIONS` column 📉

---

### 🚀 Best Practices

✅ Use PDBs for:

* Deployments
* StatefulSets
* ReplicaSets

⚠️ PDBs only apply to **controllers with replicas** (not standalone pods)

❗ Don’t set `minAvailable` too high, or **node maintenance may become impossible**.

