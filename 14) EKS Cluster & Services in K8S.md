# 📅 08-09-2025 — Part 2

## 🚀 Ways to Create an EKS Cluster
We can create an Amazon EKS cluster in 3 different ways:

1. **Using Terraform (recommended)**  
   - Use existing Terraform modules to provision EKS infrastructure.

2. **Using eksctl**  
   - A CLI tool for creating and managing EKS clusters easily.

3. **Manually (via AWS Console / UI)**  
   - Using the AWS Management Console to configure networking, IAM roles, and create the EKS cluster.

> ℹ️ Note: Tools like **kops** or **kubeadm** are alternatives, but for **EKS specifically**, we rely on Terraform, eksctl, or AWS Console.

---

## ✅ Prerequisites

### 1. Install AWS CLI
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

### 2. Install Terraform on Ubuntu
```bash
# Add HashiCorp GPG key
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

# Add Terraform repo
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

# Install Terraform
sudo apt update && sudo apt install terraform -y
terraform --version
```

### 3. Provision EKS Cluster with Terraform
```bash
git clone https://github.com/hashicorp-education/learn-terraform-provision-eks-cluster
cd learn-terraform-provision-eks-cluster

aws configure       # Configure AWS credentials
terraform init      # Initialize Terraform
terraform apply -auto-approve
```

### 4. AWS CLI Configure
```bash
aws configure
# Provide:
# - Access Key ID
# - Secret Access Key
# - Region
# - Output format (json)
```

### 5. Install kubectl
```bash
# Download kubectl binary
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Make executable
sudo chmod +x kubectl

# Move to PATH
sudo mv kubectl /usr/local/bin/

# Verify
kubectl version --client
```

**Connect cluster with kubeconfig:**
```bash
aws eks --region us-east-2 update-kubeconfig --name education-eks-4LgAJQBU
```

---

## 🟦 Pods and Services

### Why Services?
- Pods get **dynamic IPs** (every restart changes IP).
- Services provide a **stable endpoint** to access Pods.

---

## 🔹 Types of Kubernetes Services

### 1. ClusterIP
- Default type, used inside the cluster only.  
- No external access.  
- Commonly used for **microservice-to-microservice communication**.  

### 2. NodePort
- Exposes the service on all cluster nodes using a static port.  
- Port range: `30000–32767`.  
- Used for **external access in dev/test environments**.  

### 3. LoadBalancer
- Creates a **Cloud Load Balancer (e.g., AWS ELB)**.  
- Provides external access with a public endpoint.  
- Mostly used in **production**.  

### 4. ExternalName
- Maps a service inside the cluster to an **external DNS name**.  
- Useful for databases, external APIs, SaaS integrations.  

---

### Pod Definition (`pod.yml`)
```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: cluster-pod-1
  labels:
    ENV: DEV-PAYMENT
spec:
  containers:
  - name: payment-app-1
    image: nginx
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: cluster-pod-2
  labels:
    ENV: DEV-PAYMENT
spec:
  containers:
  - name: payment-app-2
    image: nginx
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: cluster-pod-3
  labels:
    ENV: DEV-PAYMENT
spec:
  containers:
  - name: payment-app-3
    image: nginx
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: cluster-pod-4
  labels:
    ENV: DEV-PAYMENT
spec:
  containers:
  - name: payment-app-4
    image: nginx
    ports:
    - containerPort: 80
```

---

### Service Definition (`service.yml`)

#### NodePort Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-payment-nodeport
spec:
  selector:
    ENV: DEV-PAYMENT
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    nodePort: 32000
    protocol: TCP
```

#### LoadBalancer Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-payment-lb
spec:
  selector:
    ENV: DEV-PAYMENT
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
```

#### ExternalName Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-external-db
spec:
  type: ExternalName
  externalName: mydb.example.com
  ports:
  - port: 3306
    protocol: TCP
```

---

## 🔹 Access Services

### NodePort
```bash
kubectl get svc
curl http://<NodePublicIP>:32000
```

### LoadBalancer
```bash
kubectl get svc
# Copy the AWS LoadBalancer endpoint
curl http://<AWS-ELB-Endpoint>
```

### ExternalName
```bash
kubectl get svc
# Inside cluster:
mysql -h svc-external-db -P 3306 -u user -p
```

---

## 🛠 Real-Time Notes
- In production, **ClusterIP** is heavily used for service-to-service communication.  
- **NodePort** is mostly for testing environments.  
- **LoadBalancer** is used in production for external access.  
- **ExternalName** is used to integrate with **external services**.  
- **Labels + Selectors** are critical for service-to-pod communication.  
- Deleting a service does not delete pods.  

---

## 🗑 Delete Commands
```bash
kubectl delete -f pod.yml
kubectl delete -f service.yml
```

---

## 🔑 Login Inside Pod
```bash
kubectl exec -it cluster-pod-3 -- bash
```
