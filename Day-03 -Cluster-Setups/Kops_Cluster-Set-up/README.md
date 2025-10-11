
````markdown
# 🌐 Kubernetes Cluster Setup on AWS with Kops - `praveens.online`

This repository provides a step-by-step guide to set up a **Kubernetes Cluster** using **Kops** on **AWS** with the domain **praveens.online**.  

---

## ✅ Prerequisites

Make sure the following tools are installed on your system:

- ⚡ **AWS CLI** – To interact with AWS services.  
- 🐳 **kubectl** – To manage Kubernetes clusters.  
- ☸️ **Kops** – To create and manage Kubernetes clusters on AWS.  

---

## 🔧 Installation Steps

### 1️⃣ Install AWS CLI
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

````

### 2️⃣ Install kubectl

```bash
sudo apt install -y kubectl
```

### 3️⃣ Install Kops

```bash
curl -LO https://github.com/kubernetes/kops/releases/download/v1.28.0/kops-linux-amd64
chmod +x kops-linux-amd64
sudo mv kops-linux-amd64 /usr/local/bin/kops
```

---

## ⚙️ Step 1: Configure AWS CLI

Run:

```bash
aws configure
```

Provide your:

* 🔑 **AWS Access Key ID**
* 🔐 **AWS Secret Access Key**
* 🌍 **Region** (e.g., `us-east-1`)
* 📄 **Output format** (e.g., `json`)

---

## 🌍 Step 2: Set Up DNS with Route 53

1. Go to **AWS Console → Route 53**.
2. Create a **Public Hosted Zone** for `praveens.online`.
3. Update your **domain registrar (GoDaddy)** with the **name servers** from Route 53.

---

## 🗄️ Step 3: Create S3 Bucket for Kops State

```bash
aws s3api create-bucket --bucket kops-state-praveens-online --region us-east-1
aws s3api put-bucket-versioning --bucket kops-state-praveens-online --versioning-configuration Status=Enabled
```

This bucket will store the cluster state.

---

## 🔑 Step 4: Generate SSH Key Pair

```bash
ssh-keygen -t rsa -b 4096
```

👉 Press **Enter** to accept the default path (`~/.ssh/id_rsa`).

---

## 🌐 Step 5: Set Environment Variables

```bash
export KOPS_STATE_STORE=s3://kops-state-praveens-online
export NAME=praveens.online
```

* `KOPS_STATE_STORE` → Path to S3 bucket where Kops stores cluster state.
* `NAME` → Domain name (`praveens.online`).

---

## ☸️ Step 6: Create the Cluster

```bash
kops create cluster \
  --name=${NAME} \
  --cloud=aws \
  --zones=us-east-1a \
  --dns-zone=${NAME} \
  --master-size=t3.medium \
  --node-size=t3.medium \
  --node-count=2 \
  --ssh-public-key=~/.ssh/id_rsa.pub
```

### ⚡ Explanation:

* `--name=${NAME}` → Cluster name (domain).
* `--cloud=aws` → Cloud provider.
* `--zones=us-east-1a` → Availability Zone.
* `--dns-zone=${NAME}` → Route 53 DNS zone.
* `--master-size` → Instance type for master node.
* `--node-size` → Instance type for worker nodes.
* `--node-count=2` → Number of worker nodes.
* `--ssh-public-key` → Public SSH key path.

---

## 🚀 Step 7: Apply Cluster Configuration

```bash
kops update cluster --name=${NAME} --yes
```

✅ The `--yes` flag automatically provisions resources.

---

## 🔍 Step 8: Validate the Cluster

```bash
kops validate cluster --name=${NAME}
```

This will check whether all nodes are up and running 🎉.

---
