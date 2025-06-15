# 🛠️ AI DevSecOps Box (PoC) – Mac mini with OrbStack + Ollama

---

## ✅ Step 1: Install Required Tools on macOS

### 1.1 Install **Ollama**

```bash
brew install ollama
```

Run your model:

```bash
ollama run deepseek-coder
```

> Exposes API at `http://localhost:11434`

---

### 1.2 Install **OrbStack**

Download from: [https://orbstack.dev](https://orbstack.dev)
Install and launch the GUI or CLI.

---

## 🧱 Step 2: Create and Configure OrbStack VM

### 2.1 Create Ubuntu VM

Limit it to 60 GB to save disk space:

```bash
orbstack vm create devsecops \
  --os ubuntu \
  --memory 8G \
  --disk 60G
```

### 2.2 Enter the VM

```bash
orbstack shell devsecops
```

---

## 🐳 Step 3: Install Docker and Compose (in VM)

```bash
sudo apt update
sudo apt install -y docker.io docker-compose
sudo systemctl enable --now docker
```

---

## 📦 Step 4: Deploy GitLab + CSGHub via Docker

### 4.1 Start GitLab CE

```bash
docker run -d \
  --hostname gitlab.local \
  --publish 8080:80 \
  --name gitlab \
  --restart always \
  gitlab/gitlab-ce:latest
```

> Access GitLab at `http://localhost:8080` (on your Mac browser)

---

### 4.2 Deploy CSGHub

```bash
git clone https://github.com/opencsg/csghub-docker.git
cd csghub-docker
docker-compose up -d
```

---

## 🤖 Step 5: Test LLM Inference via Ollama

Ollama is running on macOS. From inside the OrbStack VM:

```bash
curl http://host.docker.internal:11434/api/generate \
  -d '{"model":"deepseek-coder","prompt":"What is DevSecOps?"}'
```

✅ You should get a streaming LLM response.

---

## 🔁 Step 6: GitLab CI Integration with Ollama

### 6.1 In a GitLab project, create `.gitlab-ci.yml`:

```yaml
test_llm_inference:
  script:
    - curl http://host.docker.internal:11434/api/generate \
        -d '{"model":"deepseek-coder","prompt":"Explain CI/CD pipeline"}'
```

### 6.2 (Optional) Add GitLab Runner

```bash
docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest
```

> Register in GitLab UI for auto pipeline execution.

---

## ☸️ Step 7 (Optional): Add Kubernetes Simulation with K3s

```bash
curl -sfL https://get.k3s.io | sh -
kubectl get nodes
```

> Use `kubectl` to simulate app deployments like in production (RKE2-style).

---

## 📂 Suggested Project Structure

```
/devsecops-poc/
├── docker-compose.yml     # CSGHub
├── .gitlab-ci.yml         # LLM + CI/CD testing
├── ollama-test.sh         # Curl to test API
├── k8s/                   # (Optional) K3s manifests
└── README.md              # Setup guide
```

---

## ✅ Summary

| Component       | Environment        | Notes                      |
| --------------- | ------------------ | -------------------------- |
| Ollama (LLM)    | macOS              | Fast GPU inference (Metal) |
| GitLab + Runner | OrbStack Ubuntu VM | Full CI/CD simulation      |
| CSGHub          | OrbStack VM        | Track models and prompts   |
| K3s (optional)  | OrbStack VM        | Lightweight Kubernetes     |
