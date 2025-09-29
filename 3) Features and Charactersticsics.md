# 🚀 Kubernetes – Characteristics & Features  

---

## 1. Characteristics of Kubernetes  
👉 General qualities/nature of Kubernetes.  

### 🔹 Zero Downtime Deployment  
- K8s updates applications without interrupting users.  
- **Example:** Facebook updates login service while old version is running → users don’t face downtime.  

### 🔹 Self-Healing  
- If a pod/container fails, K8s automatically restarts or replaces it.  
- **Example:** If one food delivery server in Swiggy crashes, another one starts automatically → service continues.  

### 🔹 High Availability  
- Keeps applications running even if some nodes fail, by maintaining replicas.  
- **Example:** Even if one hotel branch closes, customers can still order from other branches.  

### 🔹 Portability & Extensibility  
- Runs on any cloud/on-prem and supports plugins/APIs.  
- **Example:** Same app runs on AWS, GCP, or your laptop without changes.  

---

## 2. Features of Kubernetes  
👉 Specific capabilities/functions provided by Kubernetes.  

### 🔹 Automated Rollouts & Rollbacks  
- Deploys new app versions gradually and reverts if they fail.  
- **Example:** If a new app update crashes, K8s switches back to the old version automatically.  

### 🔹 Horizontal Scaling (HPA)  
- Increases or decreases pods based on traffic/load.  
- **Example:** At 1 PM, 100k people order from Swiggy → Order service pods scale up automatically.  

### 🔹 Service Discovery & Load Balancing  
- Exposes pods with DNS names/IPs and distributes traffic evenly.  
- **Example:** Customers hitting facebook.com get routed to healthy servers only.  

### 🔹 Storage Orchestration  
- Attaches local/cloud storage automatically to pods.  
- **Example:** Photo upload service in Facebook gets storage mounted automatically.  

---

## 🎯 Quick One-Liners for Interview  
- **Characteristics = Qualities** → Zero downtime, Self-healing, High availability.  
- **Features = Capabilities** → Auto rollouts/rollbacks, Scaling, Service discovery.  
