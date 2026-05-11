


Also refer RUNBOOK.md

 Foreword: while this was originally made for a two VM k8s+ one additional vm(minIO), you can leave out the minio configuration and use it as a kubernetes setup adding additional worker nodes. Can spin off a cluster in a few mins. 


---

# **Kubernetes + MinIO Homelab (Vagrant, VirtualBox, Containerd, Calico, Helm)**

This repository provisions a **3‑node Kubernetes cluster** using **Vagrant + VirtualBox**, complete with:

- 1× Control Plane  
- 1× Worker Node  
- 1× MinIO Object Storage Node (with dedicated network for pod → MinIO traffic)

The environment is ideal for **Velero backup labs**, **Kubernetes learning**, and **Dockrphage‑style reproducible homelab setups**.

---

## **📦 Repository Structure**

```
.
├── k8s-CP-setup.sh          # Control plane bootstrap script
├── k8s-worker-setup.sh      # Worker node bootstrap script
├── README.md                # This file
├── Vagrantfile              # Multi-node Vagrant environment
```

---

## **🖥️ VM Topology**

| Node        | Host-only IP       | Bridged IP       | Extra Network | Purpose |
|-------------|--------------------|------------------|---------------|---------|
| control     | 192.168.56.10      | 192.168.1.50     | —             | Kubernetes API server |
| node1       | 192.168.56.11      | 192.168.1.51     | —             | Worker node |
| minio       | 192.168.56.12      | 192.168.1.52     | 10.10.10.12   | Dedicated MinIO network |

> *“Adapter 4: MinIO ONLY — Dedicated private network (fixes pod → MinIO traffic)”*

---

## **⚙️ What the Scripts Do**

### **Control Plane Script (`k8s-CP-setup.sh`)**
- Installs dependencies (`jq`, `curl`, `git`, etc.)
- Installs **containerd** + **runc**
- Configures sysctl + kernel modules
- Installs Kubernetes components (`kubeadm`, `kubelet`, `kubectl`)
- Initializes the cluster:
  ```
  sudo kubeadm init --apiserver-advertise-address=192.168.56.10 --pod-network-cidr=192.168.0.0/16
  ```
- Installs **Calico CNI**
- Installs **Helm**
- Prints the **worker join command**

---

### **Worker Node Script (`k8s-worker-setup.sh`)**
- Same containerd + Kubernetes installation steps
- Configures kubelet with node IP
- Joins the cluster using:
  ```
  sudo bash -c "<join-command>"
  # Script syntax with passing two arguments
  sudo bash k8s-worker-setup.sh 192.168.56.x "<join-command>"
  ```
- Prepares kubeconfig directory

> Citation from your file:  
> *“Copy the join command from the control plane and run this script”*

---

## **🚀 Quickstart**

### **1. Start the VMs**
```bash
vagrant up
```

### **2. SSH into the control plane**
```bash
vagrant ssh control
sudo bash /vagrant/k8s-CP-setup.sh
```

Copy the join command printed at the end.

### **3. SSH into the worker**
```bash
vagrant ssh node1
sudo bash /vagrant/k8s-worker-setup.sh 192.168.56.11 "<join-command>"
```

### **4. Verify cluster**
```bash
kubectl get nodes -o wide
```

---

## **📦 MinIO Node**

The MinIO VM is provisioned but **not automatically configured**.  
You can install MinIO manually or via Helm:

```bash
helm repo add minio https://charts.min.io/
helm install minio minio/minio -n minio --create-namespace
```

---

## **🛠️ Requirements**

- Vagrant ≥ 2.3  
- VirtualBox ≥ 7  
- 8 GB RAM minimum  
- Linux/macOS/Windows host

---

