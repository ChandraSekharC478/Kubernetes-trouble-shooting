# Pod Scheduling in Kubernetes: NodeSelector, Node Affinity, Taints & Tolerations

This README documents key Kubernetes scheduling mechanisms with practical examples and scenarios using `kind` multi-node clusters.

---

## 🛠️ KIND Cluster Setup

We used `kind` to simulate a multi-node Kubernetes cluster to test pod scheduling:

### Step 1: Install KIND
```bash
brew install kind
```

### Step 2: Create Multi-Node Cluster
```yaml
# kind-multi-node.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
  - role: worker
```
```bash
kind create cluster --name chandhu-multi-node --config kind-multi-node.yaml
```

### Step 3: Validate Nodes
```bash
kubectl get nodes
```

---

## 🚦 1. NodeSelector Example

### Scenario:
You want your pod to run only on a specific node with a given label.

### 🏷️ Step 1: Label a Node
```bash
kubectl label nodes chandhu-multi-node-worker3 node-name=arm-worker
```

### 📝 Deployment with NodeSelector
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      nodeSelector:
        node-name: arm-worker
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```
> ✅ Pod will be scheduled only on a node matching `node-name=arm-worker`

---

## 💡 2. Node Affinity Example

### Scenario:
You want more flexible rules to match nodes with specific labels.

### 📝 Deployment with Node Affinity
```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: node-name
            operator: In
            values:
            - arm-worker
```
> ✅ Node Affinity gives you richer expression-based matching over simple key/value.

### Node Affinity Types
- `requiredDuringSchedulingIgnoredDuringExecution`: **Hard rule**, if node doesn't match, pod won't schedule.
- `preferredDuringSchedulingIgnoredDuringExecution`: **Soft rule**, pod prefers matching node but can fall back.

---

## 🚫 3. Taints and Tolerations

### Scenario:
You want to **repel** pods from a node **unless** they explicitly tolerate the condition.

### Step 1: Add Taints to Nodes
```bash
kubectl taint nodes chandhu-multi-node-worker key1=value1:NoSchedule
kubectl taint nodes chandhu-multi-node-worker2 key1=value1:NoSchedule
```

### 📝 Deployment with Tolerations
```yaml
spec:
  tolerations:
    - key: "key1"
      operator: "Equal"
      value: "value1"
      effect: "NoSchedule"
```
> ✅ This allows pods to be scheduled **on tainted nodes**.

### Taint Effects:
- `NoSchedule`: New pods will not schedule unless they tolerate the taint.
- `PreferNoSchedule`: Kubernetes will try to avoid placing a pod on that node but may still do so.
- `NoExecute`: New pods won’t schedule and existing pods will be evicted unless they tolerate the taint.

### 🛑 If you remove the tolerations block:
- Pods will only be scheduled on nodes **without** the taint (`worker3` in our case).

---

## 🔍 Summary Table
| Feature              | Purpose                              | Behavior                              |
|---------------------|--------------------------------------|---------------------------------------|
| `nodeSelector`      | Simple key-value node targeting      | Hard requirement                      |
| `nodeAffinity`      | Expression-based node matching       | Flexible (preferred or required)      |
| `taints`            | Repels pods from a node              | Requires matching toleration          |
| `tolerations`       | Allows pods to run on tainted nodes  | Opt-in to taints                      |
| `NoSchedule`        | Taint effect                         | Block scheduling unless tolerated     |
| `NoExecute`         | Taint effect                         | Block and evict unless tolerated      |
| `PreferNoSchedule`  | Taint effect                         | Avoid node, but not enforced strictly |
| `preferredDuringSchedulingIgnoredDuringExecution` | Node Affinity type | Soft preference for node             |

---

## 🔧 Helpful Commands
```bash
kubectl get nodes --show-labels
kubectl describe nodes <node-name>
kubectl taint nodes <node> key=value:NoSchedule
kubectl label nodes <node> key=value
kubectl get pods -o wide
```

---

🧠 By combining these tools, you gain powerful control over **where** your pods run in your cluster — useful for workloads separation, architecture compatibility (arm vs amd), and resource isolation.