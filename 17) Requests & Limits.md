# 🔹 Requests and Limits in Kubernetes

## 1. Definition  
**Requests and Limits** in Kubernetes are resource constraints defined for CPU and Memory at the **container level** inside a Pod spec.  

- **Request** → The **minimum guaranteed** CPU/Memory a container needs.  
- **Limit (Cap)** → The **maximum allowed** CPU/Memory a container can consume.  

📌 Together, they are called **Resource Requests and Limits**.  
They ensure Pods get fair resource allocation, prevent resource hogging, and keep the cluster stable.  

---

## 2. Why do we use them?  
- **Requests** → Help the **Kubernetes scheduler** decide which node can host the Pod.  
- **Limits** → Protect nodes by **capping container usage** so one Pod doesn’t affect others.  
- Ensures **fair sharing of resources** in the cluster.  

---

## 3. Terminology / How do we call them?  
- **CPU/Memory Requests** → reserved resources (guaranteed minimum).  
- **CPU/Memory Limits** → capped resources (hard maximum).  
- Together → **Kubernetes Resource Requests and Limits**.  

📱 **Mobile Data Analogy**  
- **Request** = Minimum data guaranteed (e.g., 1 GB always available).  
- **Limit (Cap)** = Maximum data you can ever use (e.g., 2 GB).  
- If you cross the cap:  
  - Internet slows → CPU throttling.  
  - Internet stops → Memory OOMKilled.  

---

## 4. Example in YAML  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        memory: "128Mi"   # minimum memory guaranteed
        cpu: "250m"       # minimum CPU guaranteed (0.25 core)
      limits:
        memory: "256Mi"   # capped at 256Mi (hard maximum)
        cpu: "500m"       # capped at 0.5 CPU (hard maximum)
```

---

## 5. What happens if…?  
- **Pod uses < request** → Runs normally.  
- **Pod uses between request & limit** → Allowed to burst (if resources free).  
- **Pod exceeds CPU limit** → Gets **throttled** (slowed down).  
- **Pod exceeds Memory limit** → Gets **OOMKilled** (terminated).  

---

## 6. QoS (Quality of Service) Classes  
- **Guaranteed** → Requests = Limits.  
- **Burstable** → Requests < Limits.  
- **BestEffort** → No requests/limits defined.  

---

## ✅ Final Summary  
- **Requests** = Minimum guarantee (scheduler promise).  
- **Limits** = Maximum cap (runtime enforcement).  
- **Concept name** = *Kubernetes Resource Requests and Limits* (part of Kubernetes Resource Management).  
- Think of it like a **mobile data plan** → request = guaranteed GB, limit = maximum GB (cap).  
