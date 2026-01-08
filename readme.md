# Kubernetes (k3s) + Docker Setup on Windows using WSL

This document covers installing WSL, Docker, and k3s on Windows (via Ubuntu WSL) and deploying a sample Nginx application on Kubernetes.

---

## Install WSL on Windows

```bash
wsl --install
```

Installs Windows Subsystem for Linux with Ubuntu. Required to run Linux-based tooling.

---

## Install k3s (Lightweight Kubernetes)

```bash
curl -sfL https://get.k3s.io | sh -
```

Installs k3s, a lightweight Kubernetes distribution for local clusters.

---

## Verify Node Status

```bash
sudo k3s kubectl get node
```

Checks if the Kubernetes node is in `Ready` state.

---

## Check k3s Service Status

```bash
sudo systemctl status k3s
```

Verifies that the k3s service is running.

---

## Update kubeconfig Permissions

```bash
sudo chmod 777 /etc/rancher/k3s/k3s.yaml
```

Allows kubectl access without permission issues.

---

## List All Pods

```bash
kubectl get pods -A
```

Confirms cluster health by listing all pods.

---

## Install Docker on Ubuntu (WSL)

### Check Ubuntu Version

```bash
lsb_release -a
```

Identifies the Ubuntu version to ensure compatibility.

---

### Add Docker GPG Key

```bash
sudo apt update
sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Adds Docker’s official GPG key for secure package installation.

---

### Add Docker Repository

```bash
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

Configures Docker’s official APT repository.

---

### Install Docker Engine

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Installs Docker and required components.

---

## Manage Docker Service

```bash
sudo systemctl status docker
sudo systemctl start docker
sudo systemctl enable docker
```

Checks Docker status, starts the service, and enables it at boot.

---

## Docker Socket Permission & Image Pull

```bash
sudo chmod 777 /var/run/docker.sock
docker pull nginx:1.29.4
```

Allows non-root Docker usage and pulls the Nginx image.

---

## Create Dockerfile

```dockerfile
FROM nginx
COPY static-html-directory /usr/share/nginx/html
```

Defines a simple Docker image using Nginx with custom static content.

---

## Build Docker Image

```bash
docker build -t hello-world-nginx:v1 .
```

Builds a custom Nginx image.

---

## List Docker Images

```bash
docker images
```

Displays locally available Docker images.

---

## Run Docker Container

```bash
docker run --name hello-world-nginx hello-world-nginx:v1
```

Runs the container in foreground mode.

```bash
docker run --name hello-world-nginx -d -p 8080:80 hello-world-nginx:v1
```

Runs the container in detached mode, mapping host port 8080 to container port 80.

---

## Run Pod in Kubernetes (Imperative)

```bash
kubectl run hello --image=hello-world-nginx:v1
```

Creates a pod using an imperative command.

---

## List Images in k3s Container Runtime

```bash
sudo k3s ctr images list
```

Shows images available in k3s container runtime.

---

## Export and Import Docker Image to k3s

```bash
docker save hello-world-nginx:v1 -o hello-world-nginx.tar
sudo k3s ctr images import hello-world-nginx.tar
```

Transfers Docker image into the k3s runtime.

---

## Verify Imported Image

```bash
sudo k3s ctr images list | grep hello-world-nginx
```

Confirms the image is available in k3s.

---

## Run Pod Using Imported Image

```bash
kubectl run hello2 --image=hello-world-nginx:v1
kubectl describe po hello2
```

Creates and inspects the Kubernetes pod.

---

## Imperative vs Declarative

- Imperative example:
```bash
kubectl run hello --image=hello-world-nginx

```

# Ansible Control & Runner Setup on Ubuntu (WSL)

This document explains how to set up **Ansible with a control user and a separate runner user**
on Ubuntu (WSL). Each step is described clearly and can be reproduced on any fresh system.

This setup follows best practices for **separation of concerns**:
- **Control user**: runs Ansible commands
- **Runner user**: executes tasks on the managed host (localhost)

---

## 1. Create Runner User

Create a dedicated user that Ansible will manage.

```bash
sudo adduser runner
```

Add the user to the sudo group:

```bash
sudo usermod -aG sudo runner
```

---

## 2. Configure Passwordless Sudo for Runner

Ansible automation requires non-interactive sudo.

Open the sudoers file safely:

```bash
sudo visudo
```

Add this line at the **very bottom**:

```text
runner ALL=(ALL) NOPASSWD:ALL
```

### Save (Nano editor)
- CTRL + O → ENTER → CTRL + X

### Verify
```bash
su - runner
sudo whoami
```

Expected output:
```
root
```

---

## 3. Prepare Control User for Ansible

Generate SSH keys for passwordless authentication:

```bash
ssh-keygen
```

Press ENTER for all prompts.

---

## 4. Install and Enable OpenSSH Server

Update packages:

```bash
sudo apt update
```

Install OpenSSH server:

```bash
sudo apt install -y openssh-server
```

Start and enable SSH:

```bash
sudo systemctl start ssh
sudo systemctl enable ssh
sudo systemctl status ssh
```

---

## 5. Configure SSH Access to Runner

Copy the SSH key to the runner user:

```bash
ssh-copy-id runner@localhost
```

Test SSH access:

```bash
ssh runner@localhost whoami
```

Expected output:
```
runner
```

---

## 6. Install Ansible and Git on Control User

```bash
sudo apt update
sudo apt install -y ansible git
```

Verify Ansible installation:

```bash
ansible --version
```

---

## 7. Configure Ansible Inventory

Create the Ansible configuration directory:

```bash
sudo mkdir -p /etc/ansible
```

Create the hosts file:

```bash
sudo vi /etc/ansible/hosts
```

Add the following content:

```ini
[runner]
localhost ansible_user=runner ansible_connection=ssh
```

Save and exit.

---

## 8. Test Ansible Connectivity

Run an Ansible ping test:

```bash
ansible runner -m ping
```

Expected output:

```text
localhost | SUCCESS => {
    "ping": "pong"
}
```

---

## Result

You now have:
- A dedicated **runner** user with passwordless sudo
- SSH-based access from the control user
- A working Ansible inventory
- A validated Ansible control-to-runner setup

This environment is ready for installing k3s, Docker, and running CI/CD automation.

