# 🛠️ AI DevSecOps Box PoC on Mac mini (VM Name: `aidso`)

This guide walks through setting up an AI DevSecOps environment using RKE2 (Rancher Kubernetes Engine 2) on a Mac mini, with GPU acceleration via Metal.

---

## ✅ Step 1: Install Required Tools on macOS

### 1.1 Install Ollama for Local AI Inference
```bash
# Install Ollama using Homebrew
brew install ollama

# Download and run the Gemma 3 12B model (GPU-accelerated via Metal)
ollama run gemma3:12b
```

> Ollama API will be available at `http://localhost:11434`

### 1.2 Install OrbStack for Container Management
Download and install from [https://orbstack.dev](https://orbstack.dev)

---

## 🧱 Step 2: Set Up Development Environment

### 2.1 Create Ubuntu VM in OrbStack
```bash
# Create a new Ubuntu 22.04 VM with 60GB disk space
orb create ubuntu:22.04 aidso
```

### 2.2 Access the VM
```bash
# Enter the VM shell
orb shell
```

---

## 🐳 Step 3: Deploy RKE2 Kubernetes Cluster

### 3.1 Install RKE2 Server
```bash
# Download and install RKE2
curl -sfL https://get.rke2.io | sudo sh -

# Enable and start RKE2 service
sudo systemctl enable rke2-server.service
sudo systemctl start rke2-server.service
```

### 3.2 Configure Environment
```bash
# Add RKE2 binaries to PATH and set KUBECONFIG
cat >> ~/.bashrc <<'EOL'
export PATH="/var/lib/rancher/rke2/bin:$PATH"
export KUBECONFIG="/etc/rancher/rke2/rke2.yaml"
EOL

# Apply changes
source ~/.bashrc
```

### 3.3 Configure RKE2 Settings
```bash
# Set kubeconfig permissions in RKE2 config
sudo -i
cat >> /etc/rancher/rke2/config.yaml <<'EOL'
write-kubeconfig-mode: "0644"
EOL

# Configure Cilium as the Container Network Interface (CNI)
cat >> /etc/rancher/rke2/config.yaml <<'EOL'
cni: cilium
EOL
```

### 3.4 Configure Cilium Network Plugin
```bash
# Create Cilium configuration with advanced features
cat >> /var/lib/rancher/rke2/server/manifests/rke2-cilium-config.yaml <<'EOL'
apiVersion: helm.cattle.io/v1
kind: HelmChartConfig
metadata:
  name: rke2-cilium
  namespace: kube-system
spec:
  valuesContent: |-
    eni:
      enabled: true
    kubeProxyReplacement: true
    k8sServiceHost: "localhost"
    k8sServicePort: "6443"
    hubble:
      enabled: true
      relay:
        enabled: true
      ui:
        enabled: true
EOL
```

### 3.5 Restart and Verify Installation
```bash
# Restart RKE2 to apply changes
sudo systemctl restart rke2-server

# Monitor RKE2 logs
journalctl -u rke2-server -f

# Copy kubeconfig to host machine
cp /etc/rancher/rke2/rke2.yaml kubeconfig

# Exit the VM
exit
```

### 3.6 Configure Host Machine
```bash
# Set KUBECONFIG environment variable
export KUBECONFIG=kubeconfig

# Verify cluster is accessible
kubectl get nodes
```

> RKE2 Kubernetes API will be available at `https://localhost:6443`