
# 🚀 Deploy Application on E2E Kubernetes + Ingress + Route53 with ExternalDNS

---

## **Step 1: Prerequisites**

* **Kubernetes cluster** running on E2E (with `kubectl` access).
* **Domain `praveens.online`** in Route53 (public hosted zone).
* **AWS IAM user** with Route53 permissions.
* **AWS CLI + Helm** installed locally.

---

## **Step 2: Create AWS IAM User for ExternalDNS**

1. Go to **AWS Console → IAM → Users → Create User** (e.g., `external-dns-user`).
2. Attach this **custom policy**:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "route53:ChangeResourceRecordSets"
      ],
      "Resource": [
        "arn:aws:route53:::hostedzone/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "route53:ListHostedZones",
        "route53:ListResourceRecordSets"
      ],
      "Resource": ["*"]
    }
  ]
}
```

3. Save the **Access Key ID** and **Secret Key**.

---

## **Step 3: Install NGINX Ingress Controller**

Run:

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

Check:

```bash
kubectl get svc -n ingress-nginx
```

Expected:

```
ingress-nginx-controller   LoadBalancer   <PUBLIC-IP>   80:XXXX/TCP,443:YYYY/TCP
```

⚠️ Note this **PUBLIC-IP** (it will be mapped to your Route53 DNS later).

---

## **Step 4: Deploy a Sample NGINX Application**

Create file `nginx-app.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

Apply:

```bash
kubectl apply -f nginx-app.yaml
```

---

## **Step 5: Create Ingress Resource**

File: `nginx-ingress.yaml`

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: nginx.praveens.online
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 80
```

Apply:

```bash
kubectl apply -f nginx-ingress.yaml
```

---

## **Step 6: Create AWS Credentials Secret**

Save your AWS keys in Kubernetes:

```bash
kubectl create secret generic aws-credentials \
  --from-literal=aws_access_key_id=<YOUR_AWS_ACCESS_KEY> \
  --from-literal=aws_secret_access_key=<YOUR_AWS_SECRET_KEY> \
  -n kube-system
```

---

## **Step 7: RBAC for ExternalDNS**

File: `externaldns-rbac.yaml`

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: external-dns
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: external-dns
rules:
- apiGroups: [""]
  resources: ["services","endpoints","pods"]
  verbs: ["get","watch","list"]
- apiGroups: ["extensions","networking.k8s.io"]
  resources: ["ingresses"]
  verbs: ["get","watch","list"]
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["list","watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: external-dns-viewer
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: external-dns
subjects:
- kind: ServiceAccount
  name: external-dns
  namespace: kube-system
```

Apply:

```bash
kubectl apply -f externaldns-rbac.yaml
```

---

## **Step 8: Deploy ExternalDNS**

File: `externaldns.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: external-dns
  namespace: kube-system
spec:
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: external-dns
  template:
    metadata:
      labels:
        app: external-dns
    spec:
      serviceAccountName: external-dns
      containers:
      - name: external-dns
        image: k8s.gcr.io/external-dns/external-dns:v0.14.2
        args:
        - --source=service
        - --source=ingress
        - --domain-filter=praveens.online   # restrict to your domain
        - --provider=aws
        - --policy=upsert-only
        - --registry=txt
        - --txt-owner-id=e2e-cluster
        env:
        - name: AWS_REGION
          value: ap-south-1
        - name: AWS_ACCESS_KEY_ID
          valueFrom:
            secretKeyRef:
              name: aws-credentials
              key: aws_access_key_id
        - name: AWS_SECRET_ACCESS_KEY
          valueFrom:
            secretKeyRef:
              name: aws-credentials
              key: aws_secret_access_key
```

Apply:

```bash
kubectl apply -f externaldns.yaml
```

---

## **Step 9: Verify ExternalDNS**

Check pod:

```bash
kubectl get pods -n kube-system | grep external-dns
```

Check logs:

```bash
kubectl logs -n kube-system deploy/external-dns
```

Expected log:

```
INFO: Created A record: nginx.praveens.online -> <Ingress LoadBalancer IP>
```

---

## **Step 10: Test the Application**

Wait for Route53 propagation (1–5 min).
Open in browser:

```
http://nginx.praveens.online
```


---
