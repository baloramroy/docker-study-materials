# SOP: Install Docker Engine on RHEL and CentOS

## Objective

Install **Docker** Engine, configure it to **start automatically**, **verify** the installation, and perform basic **post-installation** tasks.

---

## Prerequisites

Before beginning, ensure:

* Root or `sudo` privileges are available.
* The system has **internet** connectivity.
* **Old Docker** packages (if any) are **removed**.
* `dnf` or `yum` package manager is available.

## System Preparation

### Check OS version:

- Run

  ```bash
  cat /etc/os-release
  ```

- Example output:

  ```text
  NAME="Red Hat Enterprise Linux"
  VERSION="9.5"
  ```

  or

  ```text
  NAME="CentOS Stream"
  VERSION="9"
  ```

#

### Update the System (Optional)

- Run

  ```bash
  dnf update
  ```

- Optional reboot if kernel updates were installed

  ```bash
  reboot
  ```

#

### Remove Old Docker Packages (If Present)

- Run

  ```bash
  dnf remove \
      docker \
      docker-client \
      docker-client-latest \
      docker-common \
      docker-latest \
      docker-latest-logrotate \
      docker-logrotate \
      docker-engine
  ```
  > dnf might report that you have none of these packages installed.

---

## Install using the rpm repository

### Install Required Utilities

- Run

  ```bash
  dnf install dnf-plugins-core
  ```

  > `dnf-plugins-core` package provides the commands to manage your DNF repositories.

#

### Add the Official Docker Repository

- For CentOS:

  ```bash
  dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
  ```

- For RHEL

  ```bash
  dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
  ```

- Verify:

  ```bash
  dnf repolist
  ```

- You should see something similar to:

  ```text
  docker-ce-stable
  ```

#

### Install Docker Engine

- Run and install all of this packages:

  ```bash
  dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
  ```

- If prompted to accept the GPG key:

  ```
  Is this ok [y/N]:
  ```

- Type:

  ```
  y
  ```

---

## Docker Service Management

### Start Docker Service

- Run
  
  ```bash
  systemctl start docker
  ```

- Verify:

  ```bash
  systemctl status docker
  ```

- Expected:

  ```text
  Active: active (running)
  ```

#

### Enable Docker at Boot

- Run

  ```bash
  systemctl enable docker
  ```

- Verify:

  ```bash
  systemctl is-enabled docker
  ```

- Expected:

  ```text
  enabled
  ```

#

### Verify Docker Version

- Run

  ```bash
  docker --version
  ```

- Example:

  ```text
  Docker version 28.x.x
  ```

#

### Verify Docker Can Communicate with the Daemon

- Run

  ```bash
  docker info
  ```

  >This should display server information without errors.

---

##  Run the Test Container

- Run

  ```bash
  docker run hello-world
  ```

- Expected output includes:

  ```text
  Hello from Docker!
  ```

  > This confirms Docker is functioning correctly.

---

## Allow a `Non-Root` User to Run Docker

- Create the Docker group if `docker` group doesn't exist:

  ```bash
  sudo groupadd docker
  ```

- Add your user:

  ```bash
  sudo usermod -aG docker $USER
  ```

- Apply the new group membership by logging out and back in, or start a new login shell:

  ```bash
  newgrp docker
  ```

- Test without `sudo`:

  ```bash
  docker run hello-world
  ```

---

## Lock Docker Packages:

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

## Check Docker Service

- Run to check status

  ```bash
  systemctl status docker
  ```

- Check the version:

  ```bash
  docker version
  ```

- List running containers:

  ```bash
  docker ps
  ```

- List all containers:

  ```bash
  docker ps -a
  ```

- List downloaded images:

  ```bash
  docker images
  ```

---

## Troubleshooting

### Repository not found

- Clean metadata and rebuild the cache:

  ```bash
  sudo dnf clean all
  sudo dnf makecache
  ```

### Docker service fails to start

- Inspect logs:

  ```bash
  journalctl -u docker -xe
  ```

### Permission denied when running `docker`

- Verify group membership:

  ```bash
  groups
  ```

- If `docker` is not listed, reapply:

  ```bash
  sudo usermod -aG docker $USER
  ```

  > Then log out and log back in (or use `newgrp docker`).

### Check installed Docker packages

- Run

  ```bash
  rpm -qa | grep docker
  ```

---

>[!NOTE]
This procedure provides a **clean**, **repeatable** installation suitable for **lab and production** environments on supported **RHEL and CentOS** systems.

---