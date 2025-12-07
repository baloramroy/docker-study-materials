# Docker Compose Installation

Docker Compose can be installed in **three** different ways, depending on your system and needs.

## 1. Docker Desktop (Recommended Method)
- **Best choice** for most users.
- Works on **Linux**, **Mac**, and **Windows**.
- Comes pre-packaged with:
  - **Docker Engine**
  - **Docker CLI**
  - **Docker Compose (plugin version)**
- Zero manual setup needed.
- **Check Compose version**:
  Open Docker menu → **About Docker Desktop**

## 2. Docker Compose Plugin (Linux Only)
- Use this when you already installed **Docker Engine manually** and are **not using Docker Desktop**.

- **Two ways** to install plugin:
  * **From Docker Repository** (recommended)
  * **Manual download & install**

- This installs Compose as a plugin:
  ```bash
  docker compose ...
  ```

  >**Note:** This method only works on **Linux** (not Windows or macOS).

## 3. Standalone Docker Compose (Legacy)
- **Not recommended** — for backward compatibility only.
- Old binary: Use **docker-compose** (with a hyphen)
- Used only on:
  * Linux (older systems)
  * Windows Server
    
  **Note:**
  >Use only if you must support older environments.
