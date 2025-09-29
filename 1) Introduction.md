# Kubernetes, Monolithic & Microservices – Notes

---

## 1. What is Kubernetes?  
Kubernetes (also called **K8s**) is an **open-source container orchestration platform** developed by Google.  

It is designed to automate:  
- Deployment  
- Scaling  
- Management  

of **containerized applications** across a **cluster of nodes**.  

👉 Kubernetes provides a **consistent and reliable** way to manage applications, whether they run **on-premises, or in the cloud, or in hybrid environments**.

---

## 2. Why do we use Kubernetes?  

- **Automation** → No need to manually start/stop containers.  
- **Scalability** → Easily scale apps up/down based on demand.  
- **High Availability** → If one container/node fails, Kubernetes replaces it automatically.  
- **Portability** → Runs anywhere (cloud, on-premises, hybrid).  
- **Efficiency** → Optimizes resource usage (CPU, memory).  
- **Service Discovery & Load Balancing** → Routes traffic to healthy containers automatically.  

---

## 3. Why is it called K8s?  

- The word **Kubernetes** has **10 letters**.  
- To shorten it, the middle **8 letters (ubernete)** are replaced with `8`.  

So:  
**K + 8 + s = K8s**

---

## 4. Monolithic Application  

**Definition:**  
A **monolithic application** is a **single, tightly coupled software system** where all components (Frontend, Business Logic, Database) are packaged and deployed **together as one unit**.  

- If one part fails/slows down → the entire app is affected.  
- Scaling is hard → must scale the whole system.  
- Even a **small change** requires redeploying the entire app.  

👉 **AKA:**  
A single unified software system where all modules run together.  

---

### Example: (Restaurant Story – Monolithic Application)  

Imagine **Srinivas Restaurant**:  

- The restaurant has **three servers** → Frontend, Logic, Database.  
- But they all run together as **one big application**.  

#### Scenario:  
- Customer orders food online at **1:00 PM**  
- Request → Frontend → Logic → Database  
- Response → Database → Logic → Frontend  

If **100k customers** order at once → the single server **cannot handle it** → whole system **crashes**.  

If there’s a bug (Veg Biryani → wrong dish served), fixing requires **redeploying the entire app**.  

👉 In Monolithic Model:  
- One server down = **Whole app down**  
- No flexibility/independence between modules  

---

## 5. Microservices  

**Definition:**  
A **microservices architecture** breaks an application into **small, independent services**, each handling a **specific function** (e.g., orders, payments, delivery, menu).  

- Each service runs **independently**.  
- Services communicate using **APIs**.  
- If one service fails → the whole system **doesn’t go down**.  
- Each service can be **scaled independently**.  
- Each service may use its **own database/technology**.  

---

### Example: (Srinivas Restaurant – Microservices Application)  

Redesigning Srinivas Restaurant with microservices:  

- **Menu Service** – handles dishes  
- **Order Service** – customers place orders  
- **Payment Service** – billing & payments  
- **Delivery Service** – delivery & tracking  
- **Customer Service** – customer details, feedback  

👉 Each service runs in **separate containers (microservices)**.  

#### Scenario:  
- At **1:00 PM**, 100k orders arrive.  
- **Order Service** scales up automatically.  
- **Payment Service** & **Delivery Service** continue unaffected.  
- Bug in **Menu Service** → only that service is fixed & redeployed (no full system redeploy).  
- Even if **Delivery Service** goes down → customers can still **place orders & make payments**.  

👉 In Microservices Model:  
- Independent services = **More flexibility**  
- Easy scaling = **Better performance**  

---

## 6. Monolithic vs Microservices – Facebook Example  

- **Monolithic FB Example:**  
  *If Facebook’s Login module crashes → the whole app (Signup, Newsfeed, Messaging) also goes down.*  

- **Microservices FB Example:**  
  *If Facebook’s Login Service crashes → only Login fails, but other services (Signup, Newsfeed, Messaging) still work.*  

---
