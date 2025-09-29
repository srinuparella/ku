# 🟦 Kubernetes Full Architecture

Kubernetes has two main parts:

## 1️⃣ Control Plane (Master Node)

👉 The brain of Kubernetes. Makes global decisions, stores state, and manages the cluster.

### 🔹 Components:

#### kube-apiserver
Entry point of the cluster.  
Exposes the Kubernetes API.  
All communication (kubectl, UI, other components) goes through it.  

#### etcd
Distributed key-value store.  
Stores cluster state, configs, secrets, metadata.  
Acts like the database of Kubernetes.  

#### kube-scheduler
Decides which pod runs on which node.  
Looks at CPU, memory, taints, tolerations, affinity/anti-affinity.  

#### kube-controller-manager
Runs a set of controllers (control loops) that keep the cluster in the desired state.  

**Main Controllers inside it:**
- Node Controller → Monitors node health & status.  
- Replication Controller → Ensures correct pod replicas.  
- Endpoint Controller → Maintains Service-to-Pod mapping.  *(this top 3 controllers are main, rarely used 4th controller also)*  
- Service Account Controller → Creates service accounts & API tokens.  
- Namespace Controller → Manages namespace lifecycle.  
- Job Controller → Ensures Jobs (batch tasks) complete.  
- Deployment / ReplicaSet Controller → Handles rolling updates & replicas.  
- DaemonSet Controller → Ensures pods run on all/specific nodes.  
- StatefulSet Controller → Manages stateful apps (stable IDs, storage).  
- CronJob Controller → Runs jobs on a schedule.  

#### cloud-controller-manager
Connects Kubernetes with cloud providers (AWS, Azure, GCP).  
Manages load balancers, storage volumes, node info from cloud APIs.  

---

## 2️⃣ Worker Nodes (Node Components)

👉 The hands of Kubernetes. These machines actually run the apps (pods).

### 🔹 Components:

#### kubelet
Agent running on every node.  
Talks to the Control Plane.  
Ensures pods are running properly.  

#### kube-proxy
Handles networking, routing, load balancing between pods/services.  

#### Container Runtime (Docker, containerd, CRI-O)
Runs the actual containers inside pods.  
----------------------------------------------------------------------------------------------------------------
#### Pods   (container-d)   nothing but container in docker
Smallest deployable unit in K8s.  
Abstraction over containers (can hold 1+ containers).  

#### Note :- 🔹 Pods work on top of containers.
A Pod in Kubernetes is like a wrapper (abstraction) around one or more containers.

Here’s the relationship:

Container Runtime (Docker, containerd, CRI-O, etc.) → runs the actual containers.

Pod → Kubernetes object that tells what containers to run, how to run them, and provides a shared network namespace (IP, port space) and storage volumes for those containers.

👉 So, Kubernetes doesn’t directly deal with raw containers.
Instead, it manages Pods, and inside those Pods live the actual containers (run by the container runtime).
----------------------------------------------------------------------------------------------------------------
---

## ✅ High-Level Workflow

1. User runs: `kubectl apply -f deployment.yaml`.  
2. kube-apiserver receives request → stores state in etcd.  
3. Scheduler decides the best node for new pods.  
4. kubelet on that node asks container runtime to start containers.  
5. kube-proxy ensures networking between pods/services.  
6. Controllers keep monitoring → correct drift & maintain desired state.  

-----------------------------------------------------------------------------

In Short:

Control Plane = Brain 🧠 (makes decisions, manages state)

etcd = Memory 💾 (stores cluster data)

Controllers = Nervous System ⚡ (keeps things in desired state)

Worker Nodes = Hands ✋ (run apps)

kubelet = Messenger 📢 (talks to Brain, ensures pods run)

kube-proxy = Networker 🌐 (manages communication & load balancing)

containerd = Engine 🚀 (runs the actual containers)

👉 Flow: Control Plane (Brain) → Worker Nodes (Hands) → Pods → Containers


🧠 Control Plane (Brain)  
   │  
   ├── 💾 etcd (Memory)  
   ├── ⚡ Controllers (Nervous System)  
   │  
   ▼  
✋ Worker Node (Hands)  
   ├── 📢 kubelet (Messenger to Brain)  
   ├── 🌐 kube-proxy (Networker)  
   ├── 🚀 containerd (Engine running containers)  
   │  
   ▼  
📦 Pod (Smallest Unit)  
   │  
   ▼  
🐳 Container (Actual Application)

👉 This shows flow from Brain → Hands → Pods → Containers in a simple way.
-----------------------------------------------------------------------------
