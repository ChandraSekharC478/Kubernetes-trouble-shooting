# 🎯 Canary Deployment Strategy with NGINX Ingress (Minikube Demo)

Canary deployment is a **powerful release strategy** that gradually rolls out changes to a small percentage of users before exposing it to the full production environment. It helps validate the new version (v2) alongside the stable version (v1) in a **controlled and safe** manner.

---

## 🚀 Why Canary Deployments?

Traditional deployments replace the entire application at once, which can result in **downtime** or **broken features** reaching users.

Canary strategy avoids this by:

* Routing a small % of traffic to the new version.
* Observing its behavior.
* Deciding whether to continue, rollback, or fully promote.

---

## 🧪 Demo Setup (Minikube + NGINX Ingress)

### ✅ Prerequisites:

* Minikube
* NGINX Ingress Controller installed
* kubectl

---

## 🔧 Step-by-Step Implementation

### 1. 🟢 Start Minikube & Enable Ingress

```bash
minikube start
minikube addons enable ingress
```

### 2. 🧱 Deploy Production & Canary Pods

```yaml
# production-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: production
spec:
  replicas: 1
  selector:
    matchLabels:
      app: echo
      track: stable
  template:
    metadata:
      labels:
        app: echo
        track: stable
    spec:
      containers:
      - name: echo
        image: hashicorp/http-echo
        args:
        - "-text=production"
        ports:
        - containerPort: 5678
```

```yaml
# canary-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: canary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: echo
      track: canary
  template:
    metadata:
      labels:
        app: echo
        track: canary
    spec:
      containers:
      - name: echo
        image: hashicorp/http-echo
        args:
        - "-text=canary"
        ports:
        - containerPort: 5678
```

```bash
kubectl apply -f production-deployment.yaml
kubectl apply -f canary-deployment.yaml
```

### 3. 🔁 Create Services

```yaml
# echo-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: echo
spec:
  selector:
    app: echo
  ports:
  - port: 80
    targetPort: 5678
```

```bash
kubectl apply -f echo-service.yaml
```

### 4. 🌐 Create Canary Ingress

```yaml
# base-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: echo-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: echo.prod.mydomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: echo
            port:
              number: 80
```

```yaml
# canary-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: echo-canary-ingress
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "30" # 30% traffic to canary
spec:
  rules:
  - host: echo.prod.mydomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: echo
            port:
              number: 80
```

```bash
kubectl apply -f base-ingress.yaml
kubectl apply -f canary-ingress.yaml
```

---

## 🔍 Test The Traffic Distribution

### 1. Get Minikube IP

```bash
minikube ip
```

Suppose output = `192.168.49.2`

### 2. Run curl from minikube VM

```bash
minikube ssh
```

```bash
for i in $(seq 1 10); do
  curl -s --resolve echo.prod.mydomain.com:80:192.168.49.2 echo.prod.mydomain.com | grep "Hostname";
done
```

You will see output like:

```bash
Hostname: canary-xyz
Hostname: production-abc
Hostname: canary-xyz
...
```

> Canary will receive \~30% of the requests based on your ingress weight.

📸 *See attached screenshot for visual reference of mixed routing.*

---

## ✅ Summary

| Feature         | Purpose                       |
| --------------- | ----------------------------- |
| Canary Pod      | Gradually tests new version   |
| NGINX Ingress   | Routes traffic by annotation  |
| `canary-weight` | Controls traffic distribution |
| curl + resolve  | Test via host simulation      |

This approach gives you **full control** of rollout, rollback, and testing in production traffic without full exposure.

Happy Deploying! 🚀
