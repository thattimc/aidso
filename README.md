# 🛠️ Step-by-Step Setup for AI DevSecOps Box (OrbStack + Ollama on Mac)

---

## ✅ Step 1: Install Required Tools on macOS

### 1.1 Install Ollama (host-level LLM server)

```bash
brew install ollama
ollama run deepseek-coder
```

> This exposes a local API: `http://localhost:11434`

---

### 1.2 Install OrbStack

Download from [https://orbstack.dev](https://orbstack.dev)

---

## 🧱 Step 2: Set Up OrbStack VM

### 2.1 Create Ubuntu VM

```bash
orbstack vm create devsecops --os ubuntu --memory 8G --disk 100G
```

### 2.2 Enter the VM

```bash
orbstack shell devsecops
```

---

## 🐳 Step 3: Inside VM – Install Docker + GitLab + CSGHub

### 3.1 Install Docker

```bash
sudo apt update
sudo apt install -y docker.io docker-compose
sudo systemctl enable --now docker
```

---

### 3.2 Deploy GitLab CE

```bash
docker run -d --hostname gitlab.local \
  --publish 8080:80 --publish 443:443 \
  --name gitlab \
  --restart always \
  gitlab/gitlab-ce:latest
```

* Open `http://localhost:8080` on your Mac → access GitLab

---

### 3.3 Deploy CSGHub

```bash
git clone https://github.com/opencsg/csghub-docker.git
cd csghub-docker
docker-compose up -d
```

> You may need to adjust Docker images for `arm64` if issues arise.

---

## 🤖 Step 4: Use Ollama from Inside OrbStack VM

### 4.1 Test Ollama API Access

```bash
curl http://host.docker.internal:11434/api/generate \
  -d '{"model":"deepseek-coder","prompt":"What is RKE2?"}'
```

> You should see a streaming JSON response from the macOS-hosted Ollama.

---

## 🔁 Step 5: GitLab CI Integration with Ollama

### 5.1 Create `.gitlab-ci.yml` in your test repo

```yaml
llm_test:
  script:
    - curl http://host.docker.internal:11434/api/generate \
        -d '{"model":"deepseek-coder","prompt":"Explain CI/CD pipeline"}'
```

### 5.2 Register a GitLab Runner (optional but ideal)

```bash
docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest
```

Follow prompts to register the runner in GitLab UI.

---

## ☸️ Optional Step 6: K3s for Orchestration Simulation

If you'd like to simulate Kubernetes in OrbStack:

```bash
curl -sfL https://get.k3s.io | sh -
kubectl get nodes
```

You can then deploy GitLab, CSGHub, or custom services as Kubernetes workloads.

---

## ✅ You Now Have

| Component     | Running In  | Notes                      |
| ------------- | ----------- | -------------------------- |
| Ollama LLM    | macOS Host  | GPU accelerated via Metal  |
| GitLab CE     | OrbStack VM | Accessible via port 8080   |
| CSGHub        | OrbStack VM | Model tracking UI          |
| GitLab Runner | VM          | Can call Ollama via CI     |
| Kubernetes    | (Optional)  | Via K3s inside OrbStack VM |

---
