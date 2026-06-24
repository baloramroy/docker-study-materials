# Docker Compose Standalone (Legacy) Installation

This installation is **Not recommended** — only for backward compatibility.

- Uses the **old syntax**:
  ```bash
  docker-compose up
  ```

- **Not** New syntax
  ```bash
  docker compose up
  ```

## Install the docker-compose binaries

- **Download the standalone binary**
  ```bash
  curl -SL https://github.com/docker/compose/releases/download/v5.0.0/docker-compose-linux-x86_64 \
  -o /usr/local/bin/docker-compose
  ```

- **Make it executable**
  ```bash
  chmod +x /usr/local/bin/docker-compose
  ```

- **Test if it is install or not**
  ```bash
  docker-compose version
  ```
##
**Fix if “docker-compose not found”**
- Add a symlink to a directory in your PATH, for example:
  ```bash
  sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
  ```
