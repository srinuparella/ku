# 📌 Kubernetes Annotations & Namespace

## 🔹 1. Annotations
**Definition:**
Annotations in Kubernetes are **key-value pairs** attached to objects (Pods, Deployments, Services, etc.) that store **non-identifying metadata**.

✅ **Key Points:**
- Unlike **labels**, annotations are not for filtering/selection.
- Store extra metadata like **tooling info, commit IDs, reasons for changes**.
- Commonly used by **kubectl, CI/CD, monitoring tools, or operators**.

**Example (Deployment with `kubernetes.io/change-cause` + ECR image):**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-deploy
  annotations:
    kubernetes.io/change-cause: "Updated image to latest ECR build"
spec:
  replicas: 2
  selector:
    matchLabels:
      app: payment
  template:
    metadata:
      labels:
        app: payment
    spec:
      containers:
      - name: payment-app
        image: 123456789012.dkr.ecr.us-east-1.amazonaws.com/payment:latest  # ECR image
```

👉 Run this to check rollout history with annotation:
```bash
kubectl rollout history deployment payment-deploy
```

## 🔹 2. Namespace
**Definition:**
A Namespace in Kubernetes is a **logical partition** inside a cluster used to **organize and isolate resources**.

✅ **Key Points:**
- Used to separate **dev, test, prod** environments.
- Without specifying, resources go into the **`default` namespace**.
- Helps manage RBAC (permissions), quotas, and workload isolation.

### 📌 How to Create a Namespace
```bash
kubectl create namespace dev-namespace
```

Or with YAML:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev-namespace
```

### 📌 How to Switch Namespace
```bash
kubectl config set-context --current --namespace=dev-namespace
```

👉 After switching, all `kubectl` commands default to that namespace (no need for `-n`).

## 📝 Terminology Recap
- **Annotations** → Extra metadata (e.g., `kubernetes.io/change-cause`).
- **Labels** → Identifying metadata (used for selectors/filters).
- **Namespace** → Logical partition to isolate workloads.
- **Default Namespace** → Used if not specified.
