# Kubernetes Pods & Containers -- Notes (Without Deployment)

------------------------------------------------------------------------

## 1. What is a Pod?

-   A **Pod** is the smallest deployable unit in Kubernetes.\
-   A Pod can contain **one or more containers**.\
-   Containers inside the same Pod share:
    -   **Same network namespace** → same Pod IP (communicate via
        `localhost`).\
    -   **Same storage volumes**.

👉 Common case = **1 container per pod**\
👉 Special case = **multiple containers in one pod** (sidecar pattern,
tightly coupled apps).

------------------------------------------------------------------------

## 2. Example: One container in a Pod

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: app1
spec:
  containers:
    - name: test
      image: nginx:1.25
      ports:
        - containerPort: 80
```

-   Pod name = **app1**\
-   Container name = **test**\
-   Runs nginx on port 80

Commands:

``` bash
kubectl apply -f pod.yaml
kubectl get pods
kubectl describe pod app1
```

------------------------------------------------------------------------

## 3. Example: Multiple containers in one Pod

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: app1
spec:
  containers:
    - name: test   # 1st container
      image: nginx:1.25
    - name: uat    # 2nd container
      image: httpd:2.4
```

-   Pod name = **app1**\
-   Two containers → `test` (nginx), `uat` (httpd)\
-   Both share Pod IP, so httpd can reach nginx using `localhost:80`

------------------------------------------------------------------------

## 4. What happens when we change things?

### Case A: Change **Pod name** (`app1` → `app2`)

``` yaml
metadata:
  name: app2
```

👉 Creates a **new Pod** (`app2`) with same containers.\
👉 Old Pod (`app1`) still exists unless deleted.

------------------------------------------------------------------------

### Case B: Change **Container name** (`test` → `qa`)

``` yaml
containers:
  - name: qa
    image: nginx:1.25
```

👉 **Not allowed directly**.\
- `kubectl apply` → ❌ unchanged/error.\
- Pods are **immutable** (cannot change container names in a running
pod).

✅ Solution:

``` bash
kubectl delete pod app1
kubectl apply -f pod.yaml   # creates new pod with qa container
```

------------------------------------------------------------------------

### Case C: Add a new container to existing Pod

``` yaml
containers:
  - name: test
    image: nginx:1.25
  - name: uat
    image: httpd:2.4
```

👉 **Not possible on running Pod**.\
✅ Must recreate Pod with both containers defined.

------------------------------------------------------------------------

## 5. Key Rules (Summary)

-   Pods are **immutable** → you cannot edit container list/names/images
    in a running Pod.\
-   To change anything → **delete Pod & recreate** with updated
    manifest.\
-   Changing **Pod name** = creates a completely new Pod.\
-   Multiple containers can run in one Pod, but must be defined in YAML
    from the start.

------------------------------------------------------------------------

✅ **Final One-Liner:**\
In Kubernetes, Pods can run multiple containers, but you cannot
change container names or add new containers in a running Pod. To apply
changes, delete and recreate the Pod with the modified YAML.
