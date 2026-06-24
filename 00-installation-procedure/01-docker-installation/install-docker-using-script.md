## Install Docker Using Docker Official Script

### 1. Download and Run the official Docker installation script

  This script automatically detects your **Linux distribution (CentOS, RHEL, Ubuntu, Debian)** and installs the correct Docker packages.

- **Command:**
  
  ```bash
  curl -fsSL https://get.docker.com -o install-docker.sh
  ``` 
  
- **Purpose:**\
  Downloads the official Docker installation script.

---

### 2. Run the Script

- **Command:**

  ```bash
  sh install-docker.sh 
  ```
- **Purpose:** 
  > Installs the latest stable version of Docker Engine, Docker CLI, containerd, Docker Buildx, Docker Compose plugin.

---

### 3. Enable and Start Docker Service

- **Command:**
  
  ```bash
  sudo systemctl enable docker
  sudo systemctl start docker
  ``` 
  
- **Purpose:**
  > Ensures Docker starts automatically at boot the Docker daemon immediately

---

### 4. Verify Docker Installation

- **Command:**
  
  ```bash
  docker --version
  docker info
  ```

---

### 5. Add` non-root` user to run Docker commands:

- Command:
  
  ```bash
  usermod -aG docker $USER
  ``` 

---

### 6. Lock Docker packages to prevent accidental upgrades:

- Command:

  ```bash
  dnf versionlock add \
  docker-ce \
  docker-ce-cli \
  containerd.io \
  docker-compose-plugin \
  docker-buildx-plugin \
  docker-ce-rootless-extras
  ```

- Check the version lock list
  
  ```bash
  dnf versionlock list
  ```

  > **N.B: Then log out and log in again.**

---

>[!NOTE]
You don't have to install docker compose additionally. \
Because after **docker v20.10**, docker compose comes with a docker package called **docker-compose-plugin**.

---