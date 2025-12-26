# Uninstall Docker Compose

How you uninstall Docker Compose depends on **how you installed it**.

## 1. If Docker Compose came with Docker Desktop
To remove Compose, you must **uninstall Docker Desktop itself**.

**Warning:** Uninstalling Docker Desktop removes **all Docker components**:
* Docker Engine
* Docker CLI
* Docker Compose

**Note:** Use this only if you want to remove **Docker entirely**.

## 2. If Docker Compose was installed by using Repository

- **Ubuntu / Debian**
  ```bash
  sudo apt-get remove docker-compose-plugin
  ```

- **CentOS / Fedora / RHEL (RPM-based)**
  ```bash
  sudo yum remove docker-compose-plugin
  ```

## 3. If Docker Compose was installed manually (curl download)
- **Remove user-level install**
  ```bash
  rm $DOCKER_CONFIG/cli-plugins/docker-compose
  ```

- **Remove system-wide install**
  ```
  rm /usr/local/lib/docker/cli-plugins/docker-compose
  ```

## 
**Find where Docker Compose is installed**
- If you're unsure where the plugin is located:
  ```bash
  docker info --format '{{range .ClientInfo.Plugins}}{{if eq .Name "compose"}}{{.Path}}{{end}}{{end}}'
  ```
