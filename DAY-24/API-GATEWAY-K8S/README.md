
# 🚀 Deploying Kong API Gateway on Kubernetes with SSL, Rate Limiting & Authentication

A **production-ready Kong API Gateway** setup on Kubernetes featuring:
🌍 **Automatic SSL** via Let’s Encrypt  
⚡ **Rate limiting**  
🔒 **Authentication & Authorization**  
🔄 **Multiple backend services** (NGINX + Apache)  
🧱 **Secure HTTPS routing**  

---

## 🧩 What Is an API Gateway?

An API Gateway acts as the single entry point for all client requests to your backend microservices — securely routing, authenticating, and monitoring API traffic.

Think of it as your API’s **traffic manager**.

### 💡 Example

When you open a food delivery app:
- 🧾 User Service handles login  
- 🍔 Menu Service lists restaurants  
- 💳 Payment Service manages checkout  
- 🚴 Delivery Service tracks orders  

Instead of exposing all APIs directly, **Kong Gateway**:
✅ Authenticates users  
✅ Enforces rate limits  
✅ Routes to correct backend services  
✅ Adds SSL & observability  

---

## ⚙️ Prerequisites

Ensure you have:
- ✅ A running Kubernetes cluster (EKS, Kops, Minikube, etc.)
- ✅ `kubectl` & `helm` installed
- ✅ A domain (e.g. `testkong.praveens.online`)
- ✅ DNS managed via **AWS Route 53**

---

## 🧱 Step 1: Install Kong via Helm

```bash
helm repo add kong https://charts.konghq.com
helm repo update

kubectl create namespace kong

helm install kong kong/kong \
  --namespace kong \
  --set ingressController.installCRDs=false
````

Check pods:

```bash
kubectl get pods -n kong
```

---

## 🌐 Step 2: Map Domain (Route 53)

1. Get Kong LoadBalancer external IP:

   ```bash
   kubectl get svc -n kong
   ```
2. Create a Route 53 **A Record (Alias)** pointing to that LoadBalancer:

   ```
   Name: testkong
   Type: A (Alias)
   Target: <ELB DNS>
   ```
3. Verify DNS:

   ```bash
   nslookup testkong.praveens.online
   ```

---

## 🔐 Step 3: Install Cert-Manager

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update

helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set installCRDs=true
```

---

## 📜 Step 4: Create ClusterIssuer

📄 [`kubernetes/02-clusterissuer.yaml`](kubernetes/02-clusterissuer.yaml)

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    email: itsmepraveen8140@gmail.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
      - http01:
          ingress:
            class: kong
```

Apply:

```bash
kubectl apply -f kubernetes/02-clusterissuer.yaml
```

---

## 🧩 Step 5: Deploy Backend Services

📄 [`kubernetes/03-services.yaml`](kubernetes/03-services.yaml)

Creates NGINX and Apache deployments.

---

## 🔏 Step 6: Generate SSL Certificate

📄 [`kubernetes/04-certificate.yaml`](kubernetes/04-certificate.yaml)

---

## 🚦 Step 7: Create Kong Ingress with HTTPS Routing

📄 [`kubernetes/05-ingress.yaml`](kubernetes/05-ingress.yaml)

Includes:

* `/cycle` → NGINX
* `/dm` → Apache
* TLS using Let’s Encrypt

---

## ⚡ Step 8: Enable Rate Limiting

📄 [`kubernetes/06-ratelimit.yaml`](kubernetes/06-ratelimit.yaml)

```yaml
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: echo-rate-limit
plugin: rate-limiting
config:
  minute: 5
  policy: local
```

---

## 🔑 Step 9: Enable Key Authentication

📄 [`kubernetes/07-key-auth-plugin.yaml`](kubernetes/07-key-auth-plugin.yaml)
📄 [`kubernetes/08-consumer.yaml`](kubernetes/08-consumer.yaml)

Usage:

```bash
curl -H "apikey: dev-api-key-12345" https://testkong.praveens.online/cycle
```

Unauthorized (no key):

```bash
HTTP/1.1 401 Unauthorized
{"message":"No API key found in request"}
```

---

## 🧱 Step 10: Configure ACL (Access Control)

📄 [`kubernetes/09-acl-plugin.yaml`](kubernetes/09-acl-plugin.yaml)
📄 [`kubernetes/10-consumer-group.yaml`](kubernetes/10-consumer-group.yaml)

Only consumers in `dev-group` can access.

---

## 🔄 Step 11: Per-User Rate Limit

📄 [`kubernetes/11-user-rate-limit.yaml`](kubernetes/11-user-rate-limit.yaml)

Different limits for each consumer.

---

## ⚙️ Step 12: Response Caching

📄 [`kubernetes/12-response-cache.yaml`](kubernetes/12-response-cache.yaml)

Caches responses in memory for 30s:

* `X-Cache-Status: Miss` → first request
* `X-Cache-Status: Hit` → subsequent requests

---

## 🛡️ Step 13: IP Restriction

📄 [`kubernetes/13-ipblock-plugin.yaml`](kubernetes/13-ipblock-plugin.yaml)

Restrict access by IP:

* Deny specific IPs
* Allow subnets or open ranges

---

## ✅ Step 14: Verification Checklist

| Feature            | Test                                           | Expected Result          |
| ------------------ | ---------------------------------------------- | ------------------------ |
| 🔒 SSL             | Visit `https://testkong.praveens.online/cycle` | Valid Let’s Encrypt cert |
| ⚡ Rate Limit       | Send >5 requests/min                           | `429 Too Many Requests`  |
| 🔑 Auth            | No API key                                     | `401 Unauthorized`       |
| 🧩 ACL             | Unauthorized group                             | `403 Forbidden`          |
| 🛡️ IP Restriction | Blocked IP                                     | `403 Forbidden`          |
| 🚀 Caching         | Repeat request                                 | `X-Cache-Status: Hit`    |

---

## 👨‍💻 Author

**Praveen Chinthala**
DevOps Engineer | Cloud & Kubernetes Enthusiast

💡 Passionate about building secure, scalable infra using Docker, Kubernetes, and AWS.

🔗 [LinkedIn](https://www.linkedin.com/in/praveen-chinthala28)
💻 [GitHub](https://github.com/Praveenchoudary?tab=repositories)

---

## 💙 Happy Kubernetings! 🚀

```

---

Would you like me to generate all **YAML files** (`/kubernetes/*.yaml`) with exact code and ready for commit (including file headers, names, and annotations)?  
If you say **yes**, I’ll give you each file individually (ready to upload to GitHub).
```
