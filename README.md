# 🛠️ AI DevSecOps Box PoC on Mac mini (VM Name: `aidso`)

---

## ✅ Step 1: Install Required Tools on macOS

### 1.1 Install Ollama (for GPU inference via Metal)

```bash
brew install ollama
ollama run gemma3:12b
```

> This exposes an API at `http://localhost:11434`

---

### 1.2 Install OrbStack

Download from [https://orbstack.dev](https://orbstack.dev)

---

## 🧱 Step 2: Create and Configure OrbStack VM (`aidso`)

### 2.1 Create Ubuntu VM (with 60GB disk cap)

```bash
orb create ubuntu:22.04 aidso
```

### 2.2 Enter the VM

```bash
orb shell
```

---

## 🐳 Step 3: Install RKE2 (inside `aidso` VM)

```bash
# Install RKE2
curl -sfL https://get.rke2.io | sudo sh -

# Enable and start RKE2 service
sudo systemctl enable rke2-server.service
sudo systemctl start rke2-server.service

# Add both lines to ~/.bashrc
cat >> ~/.bashrc <<'EOL'
export PATH="/var/lib/rancher/rke2/bin:$PATH"
export KUBECONFIG="/etc/rancher/rke2/rke2.yaml"
EOL

# Source ~/.bashrc
source ~/.bashrc

# Update kubeconfig permission
sudo chmod 644 /etc/rancher/rke2/rke2.yaml

# Verify installation
kubectl get nodes

# Set kubeconfig permission
sudo -i
cat >> /etc/rancher/rke2/config.yaml <<'EOL'
write-kubeconfig-mode: "0644"
EOL

# Use Cilium for CNI
sudo -i
cat >> /etc/rancher/rke2/config.yaml <<'EOL'
cni: cilium
EOL

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

# Restart RKE2
sudo systemctl restart rke2-server

# Follow the logs
journalctl -u rke2-server -f

> RKE2 will be available at `https://localhost:6443` (default Kubernetes API port)