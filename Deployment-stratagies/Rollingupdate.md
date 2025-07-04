# 🚀 Kubernetes Deployment Strategies: Rolling Update

This document explains the concept of **Rolling Updates** in Kubernetes—why it's important, how it works, and how to practice it with hands-on examples.

---

## 🎯 Why Deployment Strategies Are Needed

When upgrading an application from version **V1 to V2**, a traditional approach would uninstall V1 and then install V2. This results in **downtime (10–20 minutes)**, impacting business operations.

**To avoid this, Kubernetes supports deployment strategies** that allow you to upgrade with minimal or zero downtime.

The three most common strategies are:

1. **Rolling Update** ✅ (Default in Kubernetes)
2. **Canary Deployment**
3. **Blue-Green Deployment**

In this file, we'll focus on **Rolling Updates**.

---

## 🔄 Rolling Update Strategy

### 🔹 Concept

In a Rolling Update:

* Pods of the old version (V1) are gradually replaced with the new version (V2).
* There is **no downtime** as the service remains available during the update.

### 🔹 Key Fields

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 25%
    maxSurge: 25%
```

* `maxUnavailable: 25%` → At most 25% of pods can be unavailable during the update.
* `maxSurge: 25%` → At most 25% more pods than the desired number can be created temporarily.

> Example: For a deployment with 4 replicas:
>
> * 1 pod can be unavailable
> * 1 extra pod can be temporarily created (up to 5 running)

---

## 🧪 Rolling Update - Hands-On Demo

### Step 1: Create a Deployment

```bash
kubectl create deploy nginx --image=nginx
```

### Step 2: Scale the Deployment

```bash
kubectl scale deploy nginx --replicas=4
```

### Step 3: Trigger Rolling Update with Invalid Image (to observe failure)

```bash
kubectl set image deployment nginx nginx=nginx:broken
```

### Step 4: Watch the Pods

```bash
kubectl get pods -w
```

You’ll see that only **one replica** will fail with `ImagePullBackOff` while the remaining **3 replicas** of the original version keep serving traffic.

This demonstrates **high availability** during an upgrade.

---

## ✅ Summary

| Field            | Description                                                      |
| ---------------- | ---------------------------------------------------------------- |
| `maxUnavailable` | Maximum number of pods that can be unavailable during the update |
| `maxSurge`       | Extra pods allowed to exceed desired replicas temporarily        |
| `Rolling Update` | Default strategy in Kubernetes                                   |

---

Next, we’ll explore **Canary Deployment** and **Blue-Green Deployment** in separate files.

Stay tuned! 🌟
