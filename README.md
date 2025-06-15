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
orbstack vm create aidso \
  --os ubuntu \
  --memory 8G \
  --disk 60G
```

### 2.2 Enter the VM

```bash
orbstack shell aidso
```

---

## 🐳 Step 3: Install Docker + Compose (inside `aidso` VM)

```bash
sudo apt update
sudo apt install -y docker.io docker-compose
sudo systemctl enable --now docker
```

---

## 📦 Step 4: Deploy GitLab + CSGHub

### 4.1 Start GitLab CE

```bash
docker run -d \
  --hostname gitlab.local \
  --publish 8080:80 \
  --name gitlab \
  --restart always \
  gitlab/gitlab-ce:latest
```

> Access GitLab via `http://localhost:8080` on your Mac browser

---

### 4.2 Deploy CSGHub

```bash
git clone https://github.com/opencsg/csghub-docker.git
cd csghub-docker
docker-compose up -d
```

---

## 🤖 Step 5: Connect to Ollama from the VM

With Ollama running on the Mac host:

```bash
curl http://host.docker.internal:11434/api/generate \
  -d '{"model":"deepseek-coder","prompt":"What is DevSecOps?"}'
```

✅ You’ll get the streamed LLM output.

---

## 🔁 Step 6: GitLab CI Integration (via `.gitlab-ci.yml`)

### Sample:

```yaml
llm_test:
  script:
    - curl http://host.docker.internal:11434/api/generate \
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
