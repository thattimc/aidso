# 🛠️ AI DevSecOps Box PoC on Mac mini (VM Name: `aidso`)

---

## ✅ Step 1: Install Required Tools on macOS

### 1.1 Install Ollama (for GPU inference via Metal)

```bash
brew install ollama
ollama run deepseek-coder
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
export PATH="/var/lib/rancher/rke2/bin:$PATH"
export KUBECONFIG='/path/to/kubeconfig

# Source ~/.bashrc
source ~/.bashrc

# Verify installation
kubectl get nodes

> RKE2 will be available at `https://localhost:6443` (default Kubernetes API port)

---

## 📦 Step 4: Deploy GitLab + CSGHub

### 4.1 Install GitLab CE on VM Host

```bash
# Install required dependencies
sudo apt update
sudo apt install -y curl openssh-server ca-certificates tzdata perl

# Add GitLab package repository
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash

# Install GitLab CE
sudo EXTERNAL_URL="http://gitlab.local" apt install gitlab-ce

# Configure GitLab
sudo gitlab-ctl reconfigure

# Get initial root password
sudo cat /etc/gitlab/initial_root_password
```

> Access GitLab via `http://localhost:8080` on your Mac browser
> Note: The initial root password is shown in the output of the last command.
> Make sure to change it after first login.

---

### 4.2 Deploy CSGHub using Helm

```bash
# Add CSGHub Helm repository
helm repo add csghub https://opencsg.github.io/csghub-helm/
helm repo update

# Create namespace for CSGHub
kubectl create namespace csghub

# Install CSGHub
helm install csghub csghub/csghub -n csghub
```

---

## 🤖 Step 5: Connect to Ollama from the VM

With Ollama running on the Mac host, create a Kubernetes service to access it:

```bash
# Create a service to access Ollama
kubectl create service externalname ollama --external-name host.docker.internal --tcp=11434:11434

# Test the connection
kubectl run -it --rm test-curl --image=curlimages/curl -- sh -c 'curl http://ollama:11434/api/generate -d '"'"'{"model":"deepseek-coder","prompt":"What is DevSecOps?"}'"'"''
```

✅ You'll get the streamed LLM output.

---

## 🔁 Step 6: GitLab CI Integration (via `.gitlab-ci.yml`)

### Sample:

```yaml
llm_test:
  script:
    - curl http://ollama:11434/api/generate \
        -d '{"model":"deepseek-coder","prompt":"Explain CI/CD pipeline"}'
```

---

## 🧪 Optional Step 7: Add Kubernetes (K3s)

Inside `aidso` VM:

```bash
curl -sfL https://get.k3s.io | sh -
kubectl get nodes
```

---

## ✅ Final Setup Summary

| Component          | Runs Where | Purpose                        |
| ------------------ | ---------- | ------------------------------ |
| **Ollama**         | macOS Host | LLM inference (GPU via Metal)  |
| **GitLab CE**      | `aidso` VM | DevOps CI/CD pipeline          |
| **GitLab Runner**  | `aidso` VM | Execute prompt-based jobs      |
| **CSGHub**         | `aidso` VM | Model registry                 |
| **K3s (Optional)** | `aidso` VM | Lightweight Kubernetes runtime |

---

Would you like a downloadable zip of the full repo folder with `.gitlab-ci.yml`, `docker-compose.yml`, and `ollama-test.sh` preconfigured for `aidso`?
