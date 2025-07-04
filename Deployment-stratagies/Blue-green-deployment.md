# 📘 Blue-Green Deployment Strategy in Kubernetes

## ✅ What is Blue-Green Deployment?

Blue-Green Deployment is a release technique that reduces downtime and risk by having **two identical environments**:

* **Blue**: The current live version.
* **Green**: The new version, tested and ready.

Traffic is initially routed to **Blue**. Once Green is verified, you **switch the traffic to Green**, making it the new production. If something goes wrong, you can **instantly rollback** to Blue.

---

## 🚀 Use Case

Suppose you have version `v1` running in production (Blue), and you want to upgrade to version `v2` (Green). You create a Green environment and test it in parallel, without affecting Blue. After testing, you switch over the traffic.

---

## 🧪 Demo: Blue-Green Deployment in Kubernetes

### Step 1: Create the Blue Deployment

```bash
kubectl create deployment blue-deployment --image=nginx:1.14 --replicas=3
kubectl expose deployment blue-deployment --port=80 --target-port=80 --name=blue-service
```

### Step 2: Create the Green Deployment

```bash
kubectl create deployment green-deployment --image=nginx:1.16 --replicas=3
kubectl expose deployment green-deployment --port=80 --target-port=80 --name=green-service
```

### Step 3: Create a Common Frontend Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-service
spec:
  selector:
    app: blue-deployment  # Initially routes traffic to blue
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

Apply it with:

```bash
kubectl apply -f app-service.yaml
```

To switch traffic to Green:

```bash
kubectl patch service app-service -p '{"spec":{"selector":{"app":"green-deployment"}}}'
```

---

## ✅ How to Verify

### Check pods and endpoints

```bash
kubectl get pods -o wide
kubectl get endpoints app-service
```

### Access the service

```bash
minikube service app-service --url
curl <service-url>
```

---

## 🔁 Rollback to Blue

If something goes wrong in Green:

```bash
kubectl patch service app-service -p '{"spec":{"selector":{"app":"blue-deployment"}}}'
```

---

## 🧹 Cleanup

```bash
kubectl delete deployment blue-deployment green-deployment
kubectl delete service blue-service green-service app-service
```

---

## 📌 Summary

| Action                  | Command/Tool                                     |
| ----------------------- | ------------------------------------------------ |
| Create Blue Deployment  | `kubectl create deployment blue-deployment ...`  |
| Create Green Deployment | `kubectl create deployment green-deployment ...` |
| Switch Traffic          | `kubectl patch service`                          |
| Rollback to Blue        | `kubectl patch service`                          |

> ✅ This method is preferred when you want zero downtime and quick rollback capability during Kubernetes deployments.
