# 📘 EKS Cluster Upgrade Guide

This guide walks you through safely upgrading your EKS (Elastic Kubernetes Service) clusters with real-world steps and best practices.

---

## 🛠️ Prerequisites

1. **📌 Cordon Critical Jobs:**

   * Prevent new pods from being scheduled.
   * Coordinate with the dev team for a 2–3 hour buffer window.

2. **🔄 Version Sync:**

   * Ensure the control plane and node groups are on the **same Kubernetes version**.

3. **🧪 Test Before Prod:**

   * Always test the upgrade in **lower environments (UAT/DEV)**.
   * Buffer time of **2–3 weeks** is recommended before doing in production.

4. **📚 Understand Kubernetes Releases:**

   * Review the official release notes carefully.

5. **📡 Master Node Knowledge:**

   * Understand components like `etcd`, `kube-apiserver`, `kube-controller-manager`, etc.

6. **📶 IP Address Requirements:**

   * Ensure **at least 5 IP addresses per subnet** are available.

7. **🔧 Version Compatibility:**

   * `kubelet`, `cluster-autoscaler`, and other critical components must match the new Kubernetes version.

---

## ✅ Cluster Creation (Optional Before Upgrade)

### Create EKS Cluster (without nodegroup)

```bash
eksctl create cluster \
  --name=observability \
  --region=us-east-1 \
  --zones=us-east-1a,us-east-1b \
  --without-nodegroup
```

### Associate IAM OIDC Provider

```bash
eksctl utils associate-iam-oidc-provider \
  --region us-east-1 \
  --cluster observability \
  --approve
```

### Create Managed Node Group

```bash
eksctl create nodegroup \
  --cluster=observability \
  --region=us-east-1 \
  --name=observability-ng-private \
  --node-type=t3.medium \
  --nodes-min=2 \
  --nodes-max=3 \
  --node-volume-size=20 \
  --managed \
  --asg-access \
  --external-dns-access \
  --full-ecr-access \
  --appmesh-access \
  --alb-ingress-access \
  --node-private-networking
```

### Update kubeconfig

```bash
aws eks update-kubeconfig --name observability
```

---

## 🔼 Kubernetes Upgrade Steps

### Step 1: Upgrade Control Plane

```bash
eksctl upgrade cluster \
  --name observability \
  --region us-east-1 \
  --version 1.33 \
  --approve
```

### Step 2: Upgrade Node Groups

You can use **Rolling Updates**, **Custom AMIs**, or **Launch Templates**.

```bash
eksctl upgrade nodegroup \
  --cluster observability \
  --name observability-ng-private \
  --region us-east-1 \
  --kubernetes-version 1.33 \
  --launch-template <template-name>
```

Or recreate using custom AMI:

```bash
eksctl delete nodegroup --cluster observability --name observability-ng-private
# Then recreate nodegroup using eksctl create nodegroup
```

### Step 3: Upgrade Addons

```bash
eksctl upgrade addon \
  --name vpc-cni \
  --cluster observability \
  --region us-east-1 \
  --approve

eksctl upgrade addon --name kube-proxy --cluster observability --region us-east-1 --approve
eksctl upgrade addon --name coredns --cluster observability --region us-east-1 --approve
```

---

## ✅ Validate Upgrade Was Successful

### 1. Verify Kubernetes Version

```bash
kubectl version --short
```

* Ensure `Server Version` and `Client Version` match the upgraded version.

### 2. Check Node Group Version

```bash
kubectl get nodes -o wide
```

* Verify all nodes reflect the new version.

### 3. Check Addons

```bash
kubectl get daemonset -n kube-system
kubectl describe daemonset <name> -n kube-system
```

### 4. Run Smoke Tests

* Deploy sample pods, services, ingress, etc.
* Test logging, autoscaling, storage, and metrics.

---

## 🧹 Deletion Steps

### Delete Node Groups

```bash
eksctl delete nodegroup \
  --cluster observability \
  --name observability-ng-private \
  --region us-east-1
```

### Delete EKS Cluster

```bash
eksctl delete cluster --name observability --region us-east-1
```

---

## 📌 Summary

| Task                  | Tool/Command Used                      |
| --------------------- | -------------------------------------- |
| Create Cluster        | `eksctl create cluster`                |
| Add Node Group        | `eksctl create nodegroup`              |
| Upgrade Control Plane | `eksctl upgrade cluster`               |
| Upgrade Node Groups   | `eksctl upgrade nodegroup` or recreate |
| Upgrade Addons        | `eksctl upgrade addon`                 |
| Delete Node Group     | `eksctl delete nodegroup`              |
| Delete Cluster        | `eksctl delete cluster`                |

---

> ⚠️ Always do upgrades in UAT/Dev first, take regular backups, and notify stakeholders before production changes.

Happy Upgrading! 🚀
