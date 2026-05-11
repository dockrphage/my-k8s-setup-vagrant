# **Runbook — Kubernetes + MinIO Vagrant Cluster**

# Foreword: while this was originally made for a two VM k8s+ one additional vm(minIO), you can leave out the minio configuration and use it as a kubernetes setup adding additional worker nodes.

This runbook provides **step‑by‑step operational guidance** for provisioning, validating, troubleshooting, and maintaining the Kubernetes + MinIO Vagrant environment.

---

## **1. Provisioning**

### **1.1 Start the environment**
```bash
vagrant up
```

### **1.2 Bootstrap the control plane**
```bash
vagrant ssh control
sudo bash /vagrant/k8s-CP-setup.sh
```

Record the join command printed at the end.

### **1.3 Bootstrap the worker**
```bash
vagrant ssh node1
sudo bash /vagrant/k8s-worker-setup.sh 192.168.56.11 "<join-command>"
```

### **1.4 Validate**
```bash
kubectl get nodes
kubectl get pods -A
```

---

## **2. MinIO Node Operations**

### **2.1 SSH into MinIO VM**
```bash
vagrant ssh minio
```

### **2.2 Install MinIO (Helm)**
```bash
helm repo add minio https://charts.min.io/
helm install minio minio/minio -n minio --create-namespace
```

### **2.3 Validate**
```bash
kubectl -n minio get pods
```

---

## **3. Networking**

### **3.1 Kubernetes Pod Network**
- Calico uses: `192.168.0.0/16`

### **3.2 MetalLB (optional)**
If enabling MetalLB later:

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.5/config/manifests/metallb-native.yaml
```

---

## **4. Common Troubleshooting**

### **4.1 Node stuck in NotReady**
Check containerd:
```bash
sudo systemctl status containerd
sudo journalctl -u containerd -f
```

Restart kubelet:
```bash
sudo systemctl restart kubelet
```

### **4.2 CNI issues**
```bash
kubectl get pods -n kube-system
kubectl describe node <node>
```

### **4.3 Worker cannot join**
Regenerate token:
```bash
kubeadm token create --print-join-command
```

---

## **5. Resetting the Cluster**

### **5.1 Reset control plane**
```bash
sudo kubeadm reset -f
sudo rm -rf ~/.kube
```

### **5.2 Reset worker**
```bash
sudo kubeadm reset -f
sudo rm -rf ~/.kube
```

### **5.3 Destroy VMs**
```bash
vagrant destroy -f
```

---

## **6. Operational Notes**

- Control plane and MinIO nodes use **2 GB RAM**, workers use **1.5 GB**.
- MinIO has a **dedicated private network** (`10.10.10.12`) to avoid pod routing issues.
- Scripts automatically detect architecture (`amd64` / `arm64`).

---

## **7. Next Steps**

- Install **Velero** with MinIO backend  
- Add more worker nodes  
- Add observability stack (Prometheus, Grafana, Loki)

---

## **Follow‑up Options**

Choose what you want next:

- **Generate Velero install guide**  
- **Add MinIO Helm values.yaml**  
- **Create architecture diagram**  
- **Produce Dockrphage‑style lab version**  
