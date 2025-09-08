# 🚀 How to Install Ollama on Ubuntu 24.04 | Docker Compose

## What is ollama?
Ollama is a free, open-source tool that allows you to download and run Large Language Models (LLMs) directly on your personal computer, offering privacy, offline access, and cost savings by eliminating the need for cloud-based services. 

## 📋 What You’ll Need
Before we dive in, make sure you have:

- 🖥️ A Linux server (Ubuntu 24.04 recommended, but other distros work too)
- 🌐 A domain name pointed to your server’s IP address
- 💾 At least 8GB of RAM (16GB+ recommended for larger models)
- 🧑‍💻 Basic familiarity with the command line

This setup is perfect if you’re concerned about privacy with commercial AI services or just want the freedom to experiment with different models on your own terms.

---

## ⚡ Getting Started: Installing Docker

Update your system:
```bash
sudo apt update
````

Install Docker:

```bash
# Add Docker's official GPG key
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# Install Docker & Compose
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# Add user to Docker group
sudo usermod -aG docker ubuntu
newgrp docker
```

Verify installation:

```bash
docker --version
docker compose version
```

---

## 📂 Creating Project Structure

```bash
mkdir ollama
cd ollama
```

---

## 🛠️ Setting Up Services with Docker Compose

Create a `docker-compose.yml` file:

```yaml
version: '3.8'

services:
  ollama-webui:
    image: ghcr.io/ollama-webui/ollama-webui:main
    container_name: ollama-webui
    restart: always
    expose:
      - "8080"
    environment:
      - 'OLLAMA_API_BASE_URL=http://ollama:11434/api'
      - 'WEBUI_AUTH=false'
    depends_on:
      - ollama
   
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    restart: always
    expose:
      - "11434"
    volumes:
      - ollama_data:/root/.ollama
    command: serve

  https-portal:
    image: steveltn/https-portal:1
    ports:
      - '80:80'
      - '443:443'
    environment:
      DOMAINS: 'your-domain.com -> http://ollama-webui:8080'
      STAGE: 'production'
    volumes:
      - https-portal-data:/var/lib/https-portal
    depends_on:
      - ollama-webui

volumes:
  ollama_data:
  https-portal-data:
```

👉 Replace `your-domain.com` with your actual domain name.

### 🔎 What Each Service Does

* **Ollama** → The engine that runs AI models locally
* **Ollama WebUI** → A sleek interface to interact with your models
* **HTTPS-Portal** → Automatic SSL certificates & secure traffic

---

## 🚀 Launching the Services

Run:

```bash
docker compose up -d
```

Check containers:

```bash
docker ps
```

---

## 🌐 Accessing Your AI Assistant

* Open `https://your-domain.com` in your browser
* First load may show a test certificate (fixed after Let’s Encrypt issues a real one)
* You should see **Ollama WebUI** 🎉

---

## 🧠 Adding AI Models

### WebUI Method:

1. Go to **Models** tab
2. Click **+** to browse models
3. Select & download a model

### Command Line:

```bash
# Pull model
docker exec -it ollama ollama pull llama3

# List installed models
docker exec -it ollama ollama list

# Remove a model
docker exec -it ollama ollama rm model-name
```

👉 Start with lightweight models like `llama3` or `mistral`.

---

## 💡 Working With Your Assistant

You can:

* Create multiple chat sessions with different models
* Adjust parameters like **temperature** for creativity
* Save prompts as templates
* Upload files for analysis (model-dependent)

---

## 🛠️ Troubleshooting

Check logs:

```bash
docker logs ollama
docker logs ollama-webui
docker logs https-portal
```

**Common issues:**

* 🚨 Models fail to load → Not enough RAM
* 🔒 SSL errors → Check domain DNS
* ⏳ Timeouts → Models may take time to load

---

## 🔐 Security Tips

* ✅ Enable authentication: `WEBUI_AUTH=true` in `docker-compose.yml`
* 🔥 Configure a firewall to restrict access
* ♻️ Keep containers updated regularly

---

