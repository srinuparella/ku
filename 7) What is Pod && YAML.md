# Kubernetes Notes

## 1. How Does a User Communicate with the Control Plane/Master Node?

### Scenario
- We have one cluster. The cluster has a **Control Plane (Master Node)** and **Worker Nodes**.  
- As a DevOps engineer, to interact with the **API Server** (inside the Master Node), we use:  
  - **Code** (Java, Node.js, Python, etc.) – but DevOps engineers usually don’t write code for this.  
  - **CLI tool (`kubectl`)** – the common way.  

So as a DevOps engineer, we mainly use **`kubectl`** (Command Line Interface).  
But before learning commands, we must first understand **what a Pod is**.  

---

## 2. What is a Pod? How Many Types of Pods?

### Docker
- In Docker → you create containers directly.  
  Example: `container1`, `container2`, `container3`.  

### Kubernetes
- In Kubernetes, containers are wrapped inside **Pods**.  
- **Pod = Collection of Containers**.  
- Pods are the **smallest deployable unit** in Kubernetes.  
- A Pod has:
  - Its **own IP address**  
  - Its **own storage**  
- Inside a Pod, containers also get their own IP addresses.  

Pods are like a **small deployment/workload**.  
A deployment = something running on the server for the user.  

Example:  
```
K8s → pod[container], pod[container], pod[container]
```

---

## 3. How to Create a Pod?

There are **two ways** to create Pods:  

### 1. Imperative Mode
- Done completely through the **command line**.  
- Does **not maintain history**.  
- Used in **KodeKloud/KodeKiller playgrounds** (open-source learning environments).  
- Example command:
  ```bash
  kubectl run srinu --image=nginx
  ```
  Here:
  - `kubectl` → command  
  - `run` → create  
  - `srinu` → Pod name  
  - `--image=nginx` → container image  

Check Pod:
```bash
kubectl get po
```

- Pod status may show:
  - `Running`
  - `Error`
  - `CrashLoopBackOff` → Kubernetes self-healing mechanism  

If you delete the cluster → the Pod is also deleted.  

---

### 2. Declarative Mode
- Uses a **YAML manifest file**.  
- **History is maintained**.  
- Widely used in **organizations** (preferred way).  
- Manifest file = YAML format description of resources.  

---

## 4. `kubectl` & API Resources

- `kubectl get nodes` → check if Worker/Master nodes are available.  
✅ Definition:

API Resources = All the resource types that the Kubernetes API Server supports and can manage.
(Examples: Pod, Node, Deployment, Service, ConfigMap, Secret, etc.)
| Column         | Meaning                                                                                  |
| -------------- | ---------------------------------------------------------------------------------------- |
| **NAME**       | Full name of the resource (e.g., `pods`, `nodes`, `deployments`)                         |
| **SHORTNAMES** | Shortcut/abbreviation (e.g., `po` for pods, `cm` for configmap, `svc` for service)       |
| **APIVERSION** | The API version + group that supports this resource (e.g., `v1`, `apps/v1`)              |
| **NAMESPACED** | Whether the resource exists inside a **namespace** (`true`) or is cluster-wide (`false`) |
| **KIND**       | The kind of object (e.g., Pod, Node, Deployment)                                         |

### in below give example of kubectl api-resources

<!-- example:
kubectl api-resources

NAME          SHORTNAMES   APIVERSION     NAMESPACED   KIND
pods          po           v1             true         Pod
services      svc          v1             true         Service
configmaps    cm           v1             true         ConfigMap
nodes                      v1             false        Node
deployments   deploy       apps/v1        true         Deployment -->
<!-- 
Explanation

Pod

Full name: pods

Shortname: po

API version: v1

Namespaced: true → Pods live inside namespaces

Kind: Pod

Node

Full name: nodes

No shortname

API version: v1

Namespaced: false → Nodes are cluster-level

Kind: Node

Deployment

Full name: deployments

Shortname: deploy

API version: apps/v1

Namespaced: true → Deployment exists inside a namespace

Kind: Deployment

👉 So in short:
kubectl api-resources tells you what objects Kubernetes can create/manage and shows how to reference them (shortnames, versions, namespaces, kinds). -->
<!-- ### Fields in API Resources this is simple exmaple of api-resources 
- **Name** → resource name (e.g., node, pod)  
- **Shortnames** → shorthand (e.g., `po` = pod, `cm` = configmap)  
- **API Version** → (e.g., `apps/v1`)  
- **Namespace** → whether the resource is namespace-scoped (`true/false`)  
- **Kind** → type of resource (e.g., Pod, Deployment)   -->

<!-- Example with `grep`:  
```bash
kubectl api-resources | grep node

returnn with node resource in which version are available
``` -->

### API Versions
- API version format:
  - **Imperative**: `<version>/<group>` (e.g., `v1/apps`)  
  - **Declarative (YAML)**: `<group>/<version>` (e.g., `apps/v1`)  
- Types of versions:
  - **Alpha**
  - **Beta**
  - **Stable**  



---

## 5. YAML Format

YAML = **Key: Value** format.  

### Data Types

#### 1. Simple Text
- **Text**:
  ```yaml
  name: srinu
  name: "srinu"
  name: 'srinu'
  ```
- **Number**:
  ```yaml
  age: 26
  ```
- **Boolean**:
  ```yaml
  single: true
  single: No
  ```

#### 2. Complex Text
- **List/Array**:
  ```yaml
  Skills: ["devops", "aws", "azure"]

  Skills:
    - devops
    - aws
    - azure
  ```
- **Object/Map**:
  ```yaml
  address:
    HouseNo: 5-1359
    Village: "Addanki"
    Landmark:
      - YSR Statue
    PhoneNumbers: ["7997227675", "9490479026"]
  ```

---

## 6. Example: Pod Manifest File (YAML)

Steps:  
1. Create directory:  
   ```bash
   mkdir k8s-pod-aug
   cd k8s-pod-aug
   code .
   ```
2. Create file → `pod.yaml`  or `pod.yml`

### `pod.yaml`
```yaml
apiVersion: v1   # core group, so just v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: nginx-container
      image: nginx
```

This is a simple manifest file for understanding purposes.  
In real projects, always refer to the official Kubernetes documentation (versions may differ, e.g., v1.24 vs v1.33).  
