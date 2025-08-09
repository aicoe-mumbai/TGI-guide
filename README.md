# TGI-guide
To install TGI first install the docker
docker run --dns=8.8.8.8 --gpus all --shm-size 1g -p 8081:80 \
  -e HUGGING_FACE_HUB_TOKEN=hug_hf_iFfBZnguFMsGYFtQHBZHqUhtFuSbmIYTzO*donotchangeascopy \
  -v /absolute/path/to/volume:/data \
  ghcr.io/huggingface/text-generation-inference:3.3.4 \
  --model-id meta-llama/Llama-3.2-1B-Instruct \
  --max-input-length 512 --max-total-tokens 1024
  
curl command to use in servers
http_proxy=http://172.16.205.246:9090 \
https_proxy=http://172.16.205.246:9090 \
curl http://172.16.34.235:8080/v1/chat/completions \
    --noproxy 172.16.34.235 \
    -X POST \
    -d '{
  "model": "tgi",
  "messages": [
    {
      "role": "system",
      "content": "You are an expert assistant skilled in explaining advanced concepts across machine learning, biology, history, and ethics."
    },
    {
      "role": "user",
      "content": "Can you explain how deep learning models, such as transformers, can be applied to biological data to uncover insights about the genetic basis of diseases like cancer? Start by describing the fundamental principles of deep learning and transformers, then explain how these principles are adapted to biological sequence data such as DNA or RNA. Provide real-world examples of models or studies that have successfully used transformers in this domain. Additionally, discuss the ethical implications of using AI in biological research, particularly in terms of data privacy and the potential for AI to reinforce biases in genetic studies. Finally, suggest a strategy for integrating domain expertise (e.g., from biologists and ethicists) into AI workflows to ensure responsible and effective use of these models in healthcare and research."
    }
  ],
  "stream": false,
  "max_tokens": 1500
}' \
    -H 'Content-Type: application/json'



**Testing**
curl http://localhost:8080/v1/chat/completions \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{
    "model": "tgi",
    "messages": [
      {
        "role": "user",
        "content": "Can you explain how deep learning models, such as transformers, can be applied to biological data?"
      }
    ],
    "max_tokens": 1500,
    "stream": true
  }'





  Here’s a one-shot, end-to-end sequence of commands to **completely wipe out any prior Docker/NVIDIA Container Toolkit installs** and then **cleanly install** the latest NVIDIA Container Toolkit on Ubuntu (20.04 or later). Just copy-and-paste each block in order.

> **Before you begin**, make sure you have:
>
> * A Debian/Ubuntu system with a supported NVIDIA driver already installed
> * Internet access (direct HTTPS to `nvidia.github.io` and `download.docker.com`)
> * Sudo privileges

---

## 1. Remove any old Docker & NVIDIA toolkit packages

```bash
# Remove Docker engine and related packages
sudo apt-get remove -y docker docker-engine docker.io containerd runc \
  || true

# Remove NVIDIA Docker v1/v2 or old container toolkit
sudo apt-get remove -y nvidia-docker nvidia-docker2 nvidia-container-runtime \
  nvidia-container-toolkit \
  || true

# Purge any leftover NVIDIA repo lists
sudo rm -f /etc/apt/sources.list.d/nvidia-docker.list \
          /etc/apt/sources.list.d/nvidia-container-toolkit.list

# Clean up
sudo apt-get autoremove -y
sudo apt-get clean
```

*Allows you to start from a truly “blank slate”* ([docs.nvidia.com][1]).

---

## 2. Install Docker CE prerequisites & Docker itself

```bash
# 2.1 Update package index & install HTTPS transport
sudo apt-get update                                              
sudo apt-get install -y \
  apt-transport-https \
  ca-certificates \
  curl \
  gnupg \
  lsb-release                                                  
                                                               
# 2.2 Add Docker’s official GPG key and repo                     
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg   
                                                                        
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null            

# 2.3 Install Docker CE                                             
sudo apt-get update                                                  
sudo apt-get install -y docker-ce docker-ce-cli containerd.io       
```

*Installs the up-to-date Docker Community Edition* ([docs.nvidia.com][1]).

---

## 3. Add NVIDIA Container Toolkit repository & GPG key

```bash
# 3.1 Detect your distro (e.g. ubuntu20.04)
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)

# 3.2 Add NVIDIA’s GPG key into a keyring
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey \
  | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

# 3.3 Add the NVIDIA Container Toolkit repo (signed)
curl -sL https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list \
  | sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' \
  | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

*Sets up apt to pull `nvidia-container-toolkit` packages* ([docs.nvidia.com][1]).

---

## 4. Install NVIDIA Container Toolkit

```bash
# 4.1 Update package lists
sudo apt-get update

# 4.2 Install the toolkit
sudo apt-get install -y nvidia-container-toolkit

# 4.3 Restart Docker so it picks up the new runtime
sudo systemctl restart docker
```

*This pulls in `nvidia-container-runtime`, `nvidia-ctk`, and all dependencies* ([docs.nvidia.com][1]).

---

## 5. Configure Docker to use the NVIDIA runtime

```bash
# 5.1 Tell Docker to use NVIDIA runtime by default
sudo nvidia-ctk runtime configure --runtime=docker

# 5.2 (Optional) Verify the daemon.json entries
jq . /etc/docker/daemon.json || cat /etc/docker/daemon.json
```

*Writes the necessary `default-runtime` settings into `/etc/docker/daemon.json`* ([docs.nvidia.com][2]).

---

## 6. Verify the install

```bash
# Run a CUDA container and check GPU visibility
sudo docker run --rm --gpus all nvidia/cuda:11.6.2-base-ubuntu20.04 nvidia-smi
```

*If you see your GPU(s) listed, you’re all set!* ([docs.nvidia.com][3]).

---

**That’s it** – you’ve completely cleaned out any legacy installs and now have the **latest** NVIDIA Container Toolkit running with Docker CE.

[1]: https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/1.13.5/install-guide.html?utm_source=chatgpt.com "Installation Guide — container-toolkit 1.13.5 documentation"
[2]: https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html?utm_source=chatgpt.com "Installing the NVIDIA Container Toolkit"
[3]: https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/1.8.1/install-guide.html?utm_source=chatgpt.com "Installation Guide — container-toolkit 1.8.1 documentation"

