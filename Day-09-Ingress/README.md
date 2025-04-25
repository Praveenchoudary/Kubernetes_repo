# 🚀 Kubernetes Host-Based Routing with NGINX Ingress & GoDaddy DNS

This guide walks you through deploying two applications (`nginx` and `httpd`) on Kubernetes using **host-based routing** via NGINX Ingress. It also includes DNS setup via **GoDaddy** to route external traffic to your apps.

---

## 🌐 Domain Setup

- **Domain Name:** `praveens.online` (this is my domain name)
- **Subdomains Used:**
  - `www.praveens.online` → Routes to `nginx`
  - `httpd.praveens.online` → Routes to `httpd`

---

## 🧰 Prerequisites

- A Kubernetes cluster (Minikube, EKS, etc.)
- `kubectl` configured
- Domain purchased on GoDaddy
- Two containerized applications: `nginx` and `httpd`

---

## 🛠️ Setup Instructions

### 1️⃣ Install NGINX Ingress Controller

Apply the official manifest:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.6/deploy/static/provider/cloud/deploy.yaml


`✅ Check the status of the ingress controller:

kubectl get pods -n ingress-nginx

🌐 Get the External IP of the ingress controller:

kubectl get svc -n ingress-nginx

    Note the EXTERNAL-IP of the ingress-nginx-controller service (type: LoadBalancer). [In my case i am using Other k8s cluster i am getting IP address in case aws you will get DNS Name at that add C_NAME RECORD IN godaddy
    You’ll use this IP in GoDaddy DNS settings.

2️⃣ Configure A Records in GoDaddy DNS
Follow these steps:

    Go to https://godaddy.com and log in.

    Under My Products, select your domain: praveens.online. (this is my domain name)

    Click DNS to manage records.

    Click Add Record and enter:

Host	Type	Points to (Ingress EXTERNAL-IP)	TTL
@	A	<Ingress EXTERNAL-IP>	600
httpd	A	<Ingress EXTERNAL-IP>	600

    @ points to the root domain (www.praveens.online)

    httpd creates the subdomain (httpd.praveens.online)


# SSL/TLS Setup in Kubernetes using cert-manager & Let's Encrypt

This guide sets up HTTPS with cert-manager and Let's Encrypt for your Kubernetes application.

---

## 🌍 Domain Info

- Domain: `www.praveens.online`
- Email: `praveen@gmail.com`

---

## 📦 Tools Used

- Kubernetes
- cert-manager
- NGINX Ingress Controller
- Let's Encrypt (ACME)
  
---

## 🛠️ Steps

### 1. Install cert-manager

```bash
kubectl create namespace cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml
kubectl get pods -n cert-manager'


### 2. Create a cluster issuer.

# clusterissuer-prod.yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    email: praveen@gmail.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-prod-account-key
    solvers:
    - http01:
        ingress:
          class: nginx

kubectl apply -f clusterissuer-prod.yaml

### 2. Create a cluster issuer.

Configure ingress
