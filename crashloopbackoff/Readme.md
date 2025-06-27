# CrashLoopBackOff Troubleshooting in Kubernetes

In Kubernetes, a `CrashLoopBackOff` status means a container is starting, crashing, starting again, and crashing again repeatedly. This guide shows 3 common scenarios and how to simulate/debug them.

---

## 🐍 Sample Python App (Docker Image Setup)
We will use a simple Python app that runs on port `8000`.

### 📄 `Dockerfile`
```Dockerfile
FROM python:3.10.0-slim
WORKDIR /app
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

### ✅ Docker Build & Push
```bash
# Build with correct command
docker build -t chandra005/crashloopbackoff:v1 .
# Push to Docker Hub
docker push chandra005/crashloopbackoff:v1
```

To simulate a wrong command crash:
```Dockerfile
CMD ["python", "app1.py"]  # app1.py does not exist
```
Build and push as:
```bash
docker build -t chandra005/crashloopbackoff:v2 .
docker push chandra005/crashloopbackoff:v2
```

---

## 🧪 Scenario 1: **Wrong Command**
### 📄 `wrongcmd.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: crashloop-example
spec:
  replicas: 1
  selector:
    matchLabels:
      app: crashlooplearning
  template:
    metadata:
      labels:
        app: crashlooplearning
    spec:
      containers:
      - name: crashlooplearning
        image: chandra005/crashloopbackoff:v2
        ports:
        - containerPort: 8000
```

> ❌ Since `app1.py` doesn't exist in the image, the container will crash and enter `CrashLoopBackOff`.

---

## 🚑 Scenario 2: **Liveness Probe Failure**
### 📄 `livenessprob.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: crashloop-example
spec:
  replicas: 1
  selector:
    matchLabels:
      app: crashlooplearning
  template:
    metadata:
      labels:
        app: crashlooplearning
    spec:
      containers:
      - name: crashlooplearning
        image: chandra005/crashloopbackoff:v2
        ports:
        - containerPort: 8000
        livenessProbe:
          exec:
            command:
            - cat
            - /tmp/healthy
          initialDelaySeconds: 0
          periodSeconds: 1
```
> ❌ If `/tmp/healthy` does not exist, the liveness probe fails, restarting the container.

✅ Fix: Ensure the application creates `/tmp/healthy` or change probe method.

---

## 🧠 Scenario 3: **Out of Memory (OOMKilled)**
### 📄 `outofmemory.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: crashloop-example
spec:
  replicas: 1
  selector:
    matchLabels:
      app: crashlooplearning
  template:
    metadata:
      labels:
        app: crashlooplearning
    spec:
      containers:
      - name: crashlooplearning
        image: chandra005/crashloopbackoff:v2
        ports:
        - containerPort: 8000
        resources:
          limits:
            cpu: "25m"
            memory: "25Mi"
```
> ❌ If the container uses more than `25Mi` memory, it will be killed by Kubernetes (`OOMKilled`).

✅ Fix: Increase memory limits based on app usage.

---

## 🔍 Liveness vs Readiness Probe
| Probe         | Purpose                            | Impact when Fails               |
|---------------|-------------------------------------|----------------------------------|
| Liveness Probe | Checks if container is alive       | Container is restarted          |
| Readiness Probe| Checks if container is ready to serve traffic | Container removed from Service endpoints |

### ✅ Sample Readiness Probe
```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8000
  initialDelaySeconds: 5
  periodSeconds: 10
```

---

## ✅ Useful Commands
```bash
kubectl get pods -w
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

---

🧠 **Conclusion:**
By simulating and understanding these 3 types of failures:
- Wrong startup command
- Failed health checks
- OOMKilled due to low resource limits

You can confidently troubleshoot and resolve `CrashLoopBackOff` errors in Kubernetes.
