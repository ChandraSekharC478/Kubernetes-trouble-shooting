# 🔐 Kubernetes Network Policies: Securing Pod Communication

This `README.md` captures the practical implementation of Kubernetes Network Policies to secure Redis access in a Minikube cluster.

---

## 🧪 Scenario

We deployed a Redis database in a dedicated namespace and verified that *any* pod could access it — which is a potential security risk. We then secured it using a **NetworkPolicy** to only allow traffic from authorized pods.

---

## 🧱 Step-by-Step Implementation

### 📌 Step 1: Create Namespace
```bash
kubectl create ns secure-namespace
```

### 📌 Step 2: Deploy Redis in `secure-namespace`
```bash
kubectl apply -f db.yaml -n secure-namespace
```
Sample `db.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
spec:
  selector:
    matchLabels:
      app: redis
  replicas: 1
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:latest
        ports:
        - containerPort: 6379
```

### ✅ Redis Pod Verification
```bash
kubectl get pods -n secure-namespace -o wide
kubectl exec -it -n secure-namespace <redis-pod-name> -- redis-cli
```
> You should be connected to Redis locally at `127.0.0.1:6379`

---

## 🚨 Problem: Unrestricted Access

We deployed a dummy HTTP server (httpd) in the default namespace and were able to access Redis by IP:

```bash
redis-cli -h <redis-pod-ip>
```
> ❌ This exposes Redis to unauthorized pods.

---

## 🔐 Step 3: Apply NetworkPolicy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: redis-network-policy
  namespace: secure-namespace
spec:
  podSelector:
    matchLabels:
      app: redis
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: known-redis-member
    ports:
    - protocol: TCP
      port: 6379
```

### Apply it:
```bash
kubectl apply -f networkpolicy.yaml -n secure-namespace
```

> ✅ Only pods with label `role: known-redis-member` in the same namespace can now access Redis.

---

## 🧪 Dummy Pod Access Attempt (httpd)

### Step 1: Deploy Dummy Pod
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-deployment
spec:
  selector:
    matchLabels:
      app: httpd
  replicas: 1
  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
      - name: httpd
        image: httpd:latest
        ports:
        - containerPort: 80
```
```bash
kubectl apply -f hack.yaml
```

### Step 2: Enter the Pod and Install Redis CLI
```bash
kubectl exec -it deploy/httpd-deployment -- /bin/bash
apt update
apt-get install lsb-release curl gpg
curl -fsSL https://packages.redis.io/gpg | gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/redis.list
apt-get update
apt-get install redis
```

### Step 3: Try to Connect to Redis
```bash
redis-cli -h <redis-pod-ip>
```
> ❌ Should **fail** if pod doesn't have the label `role: known-redis-member`

---

## 📊 Summary

| Step | Description |
|------|-------------|
| 1️⃣   | Deployed Redis in a secure namespace |
| 2️⃣   | Verified local access to Redis |
| 3️⃣   | Deployed httpd dummy pod and accessed Redis (insecure) |
| 4️⃣   | Created and applied a restrictive NetworkPolicy |
| 5️⃣   | Verified access was **blocked** for unauthorized pods |

---

## ✅ Final Notes
- **Always restrict access to sensitive services** using Network Policies
- **Test** by simulating unauthorized traffic
- Use namespace + podSelector for fine-grained control

---

🔐 *Security isn't optional — it's essential in Kubernetes.*
