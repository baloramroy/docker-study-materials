# SOP: Install Docker Engine on RHEL and CentOS

## Objective

Install Docker Engine, configure it to start automatically, verify the installation, and perform basic post-installation tasks.

---

## Prerequisites

Before beginning, ensure:

* Root or `sudo` privileges are available.
* The system has internet connectivity.
* Old Docker packages (if any) are removed.
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

- Run

```bash
dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

If prompted to accept the GPG key:

```
Is this ok [y/N]:
```

Type:

```
y
```

---

# Step 6: Start Docker Service

```bash
sudo systemctl start docker
```

Verify:

```bash
sudo systemctl status docker
```

Expected:

```text
Active: active (running)
```

---

# Step 7: Enable Docker at Boot

```bash
sudo systemctl enable docker
```

Verify:

```bash
sudo systemctl is-enabled docker
```

Expected:

```text
enabled
```

---

# Step 8: Verify Docker Version

```bash
docker --version
```

Example:

```text
Docker version 28.x.x
```

---

# Step 9: Verify Docker Can Communicate with the Daemon

```bash
sudo docker info
```

This should display server information without errors.

---

# Step 10: Run the Test Container

```bash
sudo docker run hello-world
```

Expected output includes:

```text
Hello from Docker!
```

This confirms Docker is functioning correctly.

---

# Step 11: Allow a Non-Root User to Run Docker (Optional)

Create the Docker group if necessary:

```bash
sudo groupadd docker
```

Add your user:

```bash
sudo usermod -aG docker $USER
```

Apply the new group membership by logging out and back in, or start a new login shell:

```bash
newgrp docker
```

Test without `sudo`:

```bash
docker run hello-world
```

---

# Step 12: Check Docker Service

```bash
systemctl status docker
```

Check the version:

```bash
docker version
```

List running containers:

```bash
docker ps
```

List all containers:

```bash
docker ps -a
```

List downloaded images:

```bash
docker images
```

---

# Step 13: Configure Firewall (If Exposing Container Ports)

Example: allow HTTP traffic.

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload
```

For a custom application port (example: `8080`):

```bash
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```

Verify:

```bash
sudo firewall-cmd --list-all
```

---

# Step 14: Useful Docker Service Commands

Start Docker:

```bash
sudo systemctl start docker
```

Stop Docker:

```bash
sudo systemctl stop docker
```

Restart Docker:

```bash
sudo systemctl restart docker
```

Reload configuration:

```bash
sudo systemctl reload docker
```

Enable at boot:

```bash
sudo systemctl enable docker
```

Disable at boot:

```bash
sudo systemctl disable docker
```

Check status:

```bash
sudo systemctl status docker
```

---

# Step 15: Basic Docker Validation

Check Docker version:

```bash
docker version
```

Check Docker information:

```bash
docker info
```

List images:

```bash
docker images
```

List running containers:

```bash
docker ps
```

Run an interactive container:

```bash
docker run -it alpine sh
```

Exit the container:

```sh
exit
```

---

# Troubleshooting

## Repository not found

Clean metadata and rebuild the cache:

```bash
sudo dnf clean all
sudo dnf makecache
```

## Docker service fails to start

Inspect logs:

```bash
sudo journalctl -u docker -xe
```

## Permission denied when running `docker`

Verify group membership:

```bash
groups
```

If `docker` is not listed, reapply:

```bash
sudo usermod -aG docker $USER
```

Then log out and log back in (or use `newgrp docker`).

## Check installed Docker packages

```bash
rpm -qa | grep docker
```

---

# Installation Verification Checklist

| Check             | Command                       | Expected Result          |
| ----------------- | ----------------------------- | ------------------------ |
| Docker installed  | `docker --version`            | Version displayed        |
| Service running   | `systemctl status docker`     | `active (running)`       |
| Starts on boot    | `systemctl is-enabled docker` | `enabled`                |
| Daemon accessible | `docker info`                 | Server information shown |
| Test container    | `docker run hello-world`      | Success message printed  |
| Compose plugin    | `docker compose version`      | Version displayed        |

This procedure provides a clean, repeatable installation suitable for lab and production environments on supported RHEL and CentOS systems.
