# Docker Storage Explanation

## 1. Docker Storage Concept Clasification
Docker has **two** types of storage concepts, 
1. Container data persistence
2. Daemon storage backends

---

### 1.1 Container Data Persistence

- **What it is:**\
  How **application data is saved inside a container so it does not disappear** when a container stops or is deleted.

- **Why it exists:**\
  Containers are **short-lived by default**. Without persistence, all data inside a container is lost.

- **How Docker provides it:**

  1. **Volumes** – Docker-managed persistent storage
  2. **Bind mounts** – Host directory mounted into a container
  3. **tmpfs** – In-memory storage (data lost on container stop)

- **Example use cases:**
  - Databases (MySQL, PostgreSQL)
  - Application uploads
  - Logs
  - Configuration files

**One-line summary:**\
*Container data persistence is about keeping application data safe beyond the container lifecycle.*

---

### 1.2 Daemon Storage Backends

- **What it is:**\
  How the Docker daemon stores **container images** and **container filesystem data** on the host using a **storage driver**.

  This includes **read-only image layers** and the **container’s temporary writable layer**, but not **persistent volume data**.

- **Container Filesystem**\
  The container filesystem is the layered, temporary filesystem created from **image layers** and a **writable layer** that the container uses while running.

  Example directories:\
  `/bin`, `/etc`, `/usr`, `/var`, `/app`

  >To the container, this looks like a normal **Linux filesystem**.


- **What lives inside the container filesystem?**

  Examples:

  * System binaries (`/bin/bash`)
  * Config files (`/etc/nginx/nginx.conf`)
  * Application code (`/app/main.py`)
  * Logs (`/var/log/app.log`) *if not mounted to a volume*

  >By default, **all these files live in image layers or the writable layer**.

- **Why it exists:**

  Docker needs a backend filesystem/driver to efficiently manage:
  - Image layers
  - Container writable layers
  - Copy-on-write operations

- **Common storage drivers:**
  - **overlay2** (default & recommended)
  - aufs (deprecated)
  - devicemapper
  - btrfs
  - zfs


- **Example:**
  ```bash
  docker info | grep "Storage Driver"
  ```

**One-line summary:**\
*Daemon storage backends control how Docker internally stores and manages images and containers filesystem.*

---

To understand why **persistent storage** is necessary, we must first understand **how Docker stores data inside a container**.

## 2. How Container Storage Works

Docker containers use a **layered** filesystem:

**Read-only Image Layers:**
- Every container starts from **read-only image layers**.
- These layers contain:
  - OS files
  - Application binaries
  - Libraries

**Writeable Container Layers:**
- Docker adds a **writable layer** on top of those **image layers** (unique for each container).
- This writable layer is:
  - Unique per container
  - Created at container start
  - Removed when the container is deleted

**Limitations of the Writable Layer**
- Any file you **create** inside the **container** goes into the **writable** layer.
- If the container is **deleted**, all data inside the writable layer is **lost**.
- It is **difficult** to **copy or move** writable-layer data to another container or to the host.


Because of this **limitation** and the **short-lived nature of writable layers**, Docker **cannot rely** on the **writable layer** for important data. Exactly for this reason Docker provides **data persistent storage**.

Persistent storage **bypasses** the **writable layer** and stores data:
- "**Outside the container lifecycle**"
- "**Independently of image rebuilds**"

## 3. Container Storage Layers Diagram

```
Container Filesystem (managed by Docker)
----------------------------------------
| Writable Layer   |  ← Temporary, unique per container
|-----------------|
| Image Layer 3    |  ← Read-only
| Image Layer 2    |  ← Read-only
| Image Layer 1    |  ← Read-only
----------------------------------------

Persistent Storage (Volumes / Bind mounts)
------------------------------------------
Stored outside the container lifecycle
Survives container removal or recreation
```