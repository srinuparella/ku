# 📅 29-08-2025

## 🟦 To Create Pod in Declarative Way (Manifest File)

### Manifest File
First of all we need some **basic core arguments/fields** for creating the manifest file, those are:

- **apiVersion**: `<apiversion/group>`
- **kind**: `<resource/workloads>` → nothing but pod name
- **metadata**: i.e. more information or additional data  
  Example: in `metadata` → (name, namespace, annotations → imp concept, Endpoint) (we will learn further)

<!-- 👉 Example comparison: 
👉 In Docker we create a container by using:

- **Docker** → `docker container run -d -P --name nginx`
- **Kubernetes Pod** → via manifest file -->

 
- **spec**: what do you want (which container you want to run inside spec)  
  Example: image, port numbers, CPU utilizations, labels and selectors

- **status**: not used by us → nothing but output (whenever pod is created, the pod output will be shown in status)

---

## 📌 Agenda
- Again install self-hosted container by installing **kubeadm cluster**
- Create one **basic pod**
- Explain **how many types in pod and how many containers**
- Explain **how to communicate pod to pod**

---

# 1️⃣ Install Self-Hosted (kubeadm cluster)

### Steps:
1. Go to **AWS Account**
2. Select **Launch Instance**
3. Create **3 instances** at a time
4. Basic requirement: `2 vCPU` and `4GB RAM` each machine
5. 3 machines → one is **control plane**, remaining two are **worker nodes**
6. Select `Ubuntu 24.04 LTS`  
   - LTS = Long Time Supported  
   - HVM = Hyperwise Virtual Machine
7. Select **instance type** = `t2.medium`
8. Select **key pair**
9. Select **security group (SG)** → ports same for all nodes  
   Increase the **volume to 20**
10. Launch instance → now 3 machines are created  and give names like this 
    - `masternode`  
    - `worknode-1`  
    - `worknode-2`
11. Login to 3 instances:
    ```bash
    ssh ubuntu@<instance-ip>
    ```
12. Run:
    ```bash
    sudo apt update
    ```
    (⚠️ Note: in organizations don’t use `sudo apt update`. Using this command may risk your job.)

---

### Install Docker on 3 VMs
1. Go to docker script and install docker in 3 VMs.
2. Run:
   ```bash
   sudo sh install-docker.sh
   ```
3. Check:
   ```bash
   docker info
   ```
   → If showing client only and **permission denied**, fix it:
   ```bash
   sudo usermod -aG docker ubuntu
   exit
   ```
   Relogin and check:
   ```bash
   docker info
   ```
   → Now showing `var/lib/docker` (root directory for docker).

---

### Install CRI-Dockerd
On all 3 VMs:
```bash
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.4.0/cri-dockerd_0.4.0.3-0.ubuntu-jammy_amd64.deb 
sudo dpkg -i cri-dockerd_0.4.0.3-0.ubuntu-jammy_amd64.deb
```

---

### Install Additional Ubuntu Packages + Containerd
```bash
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates 
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/docker.gpg 
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" 
sudo apt update 
sudo apt install -y containerd.io 
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1 
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml 
sudo systemctl restart containerd 
sudo systemctl enable containerd 
```

---

### Install kubeadm, kubelet, kubectl
- `kubeadm` → multi-node cluster  
- `kubelet` → cluster  
- `kubectl` → command line  

```bash
sudo apt-get install -y apt-transport-https ca-certificates curl gpg 
sudo mkdir -p -m 755 /etc/apt/keyrings 
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update 
sudo apt-get install -y kubelet kubeadm kubectl 
sudo apt-mark hold kubelet kubeadm kubectl
```

---

### Initialize Master Node
Before init, switch to root user:
```bash
sudo -i
```

Run init:
```bash
kubeadm init --pod-network-cidr=10.244.0.0/16 --cri-socket=unix:///var/run/cri-dockerd.sock
```

- Socket ensures Docker host & kubeadm communication.
- `.kube` folder is created with certificates, config file & join token.

👉 Copy the **token** means join command now open in notepad and save it (add CRI socket path like \ ).  

Exit root and run configure command on master node.  
Then run **join commands** on both worker nodes in root mode.

Check nodes:
```bash
kubectl get nodes -w
```

---

### Why Not Ready? (CNI issue)
We installed only kubeadm, kubelet, kubectl and docker.  
But **CNI (Container Network Interface)** is missing.  
Common CNIs:
- Weave
- Flannel
- Calico

---

### Install Flannel CNI
```bash
kubectl apply -f https://github.com/coreos/flannel/raw/master/Documentation/kube-flannel.yml
```

Check again:
```bash
kubectl get nodes -w
```
→ After few seconds, nodes become **Ready** ✅

---

# 2️⃣ Pod Creation in Declarative Mode (Manifest File)

👉 While creating pod using manifest file:

- `apiVersion: version/group`  
  If group name is core, don’t mention it.

### Example Manifest
```yaml
apiVersion: v1
kind: pod
metadata:
  name: test-pod
spec:
  containers:
    - image: httpd:latest   # apache image aka httpd
      name: my-testimage
      ports:
        - containerPort: 80
          protocol: TCP
```

### Steps
1. Save file:
   ```bash
   vi pod-test.yml
   ```
2. Run file:
   ```bash
   kubectl apply -f pod-test.yml
   ```
3. Check:
   ```bash
   kubectl get pods
   kubectl get pods -w
   ```
4. Describe:
   ```bash
   kubectl describe pod test-pod
   ```

👉 If error (CNI issue) or re-run weave/flannel.  

👉 Use quotes for image tags:
```yaml
image: "httpd:latest"
```

---

# 3️⃣ Container Types
1. Main containers
2. Sidecar containers
3. Init containers
4. Ambassador containers

💡 *Interview*: Explain all four.

---

# 4️⃣ Types of Pods
1. Single Pod
2. Multiple Pod
3. Static Pod (⭐ very important)
4. Mirror Pod (⭐ very important)

---

📌 Later: How to communicate pod-to-pod
