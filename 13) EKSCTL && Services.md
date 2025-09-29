# 🚀 Create EKS Cluster with Terraform

**Date:** 08-09-2025

---

## Part 1 → Create EKS cluster with Terraform (including AWS Configure section properly)

### Prerequisites
- An EC2 Ubuntu machine (or any machine where you can run AWS CLI & Terraform)
- IAM user with AdministratorAccess and access key + secret (or EC2 instance role with permissions)

---

## Step 1: Create EC2 Machine
1. Launch an **EC2 instance** (Ubuntu).
2. SSH login to VM:

```bash
ssh -i "key.pem" ubuntu@<public-ip>
```

---

## Step 2: Update Packages

```bash
sudo apt update
```

---

## Step 3: Install Required Tools

We need:
- AWS CLI
- Terraform
- kubectl (later)

### Install AWS CLI

```bash
sudo apt install unzip -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

### Install Terraform

```bash
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform -y
terraform --version
```

---

## Step 4: Configure AWS CLI

Run:

```bash
aws configure
```

It will prompt for:
- AWS Access Key ID
- AWS Secret Access Key
- Default region → `ap-south-1` (or your choice)
- Default output format → `json`

👉 Notes:
- For a new VM: Create an IAM User → Attach `AdministratorAccess` → Generate Access Key & Secret Key → Use them here.
- For existing VM with IAM Role: No need to configure keys. Use instance profile.

---

## Step 5: Create EKS Cluster with Terraform

Two approaches:
- `aws_eks_cluster` resource (manual) ❌
- Terraform EKS module (recommended) ✅

We'll use an example repo (Terraform EKS module / learning repo):

```bash
git clone https://github.com/hashicorp-education/learn-terraform-provision-eks-cluster
cd learn-terraform-provision-eks-cluster

terraform init
terraform apply -auto-approve
```

This will create an EKS Cluster + Worker Nodes (as defined by the repo/module).

Check cluster in AWS Console → EKS.

---

## Step 6: Install kubectl on Ubuntu

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

---

## Step 7: Connect Cluster

If `kubectl get nodes` gives an error, update kubeconfig:

```bash
aws eks --region <your-region> update-kubeconfig --name <cluster-name>
kubectl get nodes
```

Now nodes should be visible. ✅

---

## Step 8: Deploy Pods & Service (Example)

`pod.yml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cluster-pod-1
  labels:
    app: payment-app
spec:
  containers:
    - name: app-image-1
      image: nginx
      ports:
        - containerPort: 80
          protocol: TCP
---
apiVersion: v1
kind: Pod
metadata:
  name: cluster-pod-2
  labels:
    app: payment-app
spec:
  containers:
    - name: app-image-2
      image: nginx
      ports:
        - containerPort: 80
          protocol: TCP
---
apiVersion: v1
kind: Pod
metadata:
  name: cluster-pod-3
  labels:
    app: payment-app
spec:
  containers:
    - name: app-image-3
      image: nginx
      ports:
        - containerPort: 80
          protocol: TCP
---
apiVersion: v1
kind: Pod
metadata:
  name: cluster-pod-4
  labels:
    app: payment-app
spec:
  containers:
    - name: app-image-4
      image: nginx
      ports:
        - containerPort: 80
          protocol: TCP
```

`service.yml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  selector:
    app: payment-app
  type: ClusterIP
  ports:
    - port: 80
      protocol: TCP
```

Apply resources:

```bash
kubectl apply -f pod.yml
kubectl apply -f service.yml
kubectl get pods
kubectl get svc
```

Test internal communication:

```bash
kubectl exec -it cluster-pod-1 -- bash
curl http://<ClusterIP>:80
```

This should display the Nginx default page.

For external access use `NodePort` / `LoadBalancer` / `ExternalName` as required.

---

## Step 9: Delete Cluster

```bash
terraform destroy -auto-approve
```

---

# 🟦 Kubernetes Services & Scenarios

**Why Services?**
Pods have dynamic IPs. When Pods restart they get new IPs. Service provides a stable endpoint.

### 🔹 Scenario 1: Payment Application with Multiple Pods

**Application:** HDFC Bank Payment Service.

Running on 4 Pods with label `app=web` (example IPs):
- Pod1 → 10.0.0.0
- Pod2 → 10.1.0.0
- Pod3 → 10.2.0.0
- Pod4 → 10.3.0.0

**Problem:** Pod IPs change on restart → Users cannot reliably access the payment page.

**Solution:** Create a Service that selects `app=web`. The Service gives a stable endpoint (ClusterIP / NodePort / LoadBalancer).

### 🔹 Scenario 2: Pod Restart (Dynamic IP Issue)

If one Pod goes down, Kubernetes creates a replacement Pod with a new IP. Without a Service, clients cannot reach the app consistently. Service provides stable DNS and load balancing.

---

## 🔹 Types of Services

- **ClusterIP** - Default. Internal-only access (within cluster).
- **NodePort** - Exposes service on a port on each node. Access via `<NodeIP>:<NodePort>`.
- **LoadBalancer** - Uses cloud provider LB (e.g., AWS ELB) for external access. Production-friendly.
- **ExternalName** - Maps the service to an external DNS name (CNAME-style).

✅ Recommendations:
- Use **ClusterIP** for internal pod-to-pod communication.
- Use **LoadBalancer** (or NodePort + external LB) for external traffic in cloud environments.

---

## Quick Examples

### NodePort example (service-nodeport.yml):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: payment-nodeport
spec:
  type: NodePort
  selector:
    app: payment-app
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

### LoadBalancer example (service-lb.yml):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: payment-lb
spec:
  type: LoadBalancer
  selector:
    app: payment-app
  ports:
    - port: 80
      targetPort: 80
```

---

## Notes & Best Practices
- Keep AWS credentials secure. Prefer IAM roles for EC2 where possible.
- Use Terraform modules for repeatable infra (EKS module is recommended).
- Use `kubectl apply -f` for manifests; use deployments + replicasets for production instead of raw Pods.
- Use `kubectl port-forward` if you need quick local access to a pod for debugging.
- Monitor costs: EKS + EC2 + LoadBalancers incur AWS charges.

---

## Cleanup
- `kubectl delete -f <manifest>` to remove k8s resources.
- `terraform destroy -auto-approve` to remove infra created by terraform.

---

*Prepared and exported as a markdown (.md) document on request.*

