`--mount` vs `-v / --volume`
# Docker Volume Mounting Guide

## Big Picture

Docker supports **three mount types**:

1. **Volume** (Docker-managed)
1. **Bind mount** (Host-path based)
2. **tmpfs** (Memory-only)

Both `--mount` and `-v` are **just syntaxes** to attach these mounts to a container.

---

## 1. `--mount` Option (Recommended, Explicit)

**General Syntax**

```bash
docker run \
  --mount type=<mount_type>,source=<src>,target=<dst>[,options] \
  IMAGE
```

**Key Fields**

| Field     | Meaning                         |
| --------- | ------------------------------- |
| `type`    | volume / bind / tmpfs           |
| `source`  | Volume name or host path        |
| `target`  | Container path                  |
| `options` | ro, rw, propagation, tmpfs-size |

---

### 1.1 Mounting a Named Volume Using `--mount`

- **Named Volume**

  ```bash
  docker run -d \
    --mount type=volume,source=db_data,target=/var/lib/mysql \
    mysql
  ```

  **Purpose**
  - Persistent storage
  - Docker manages volume lifecycle

  **Output:** This will mount the **app_config** volume readonly at `/etc/app` in the container.


- **Read-Only Named Volume**

  ```bash
  docker run \
    --mount type=volume,source=app_config,target=/etc/app,readonly \
    myapp
  ```

  **Output:** This will mount the **app_config** volume readonly at `/etc/app` in the container.

---

## 1.2 Bind Mount Using `--mount`

- **Basic Bind Mount**

  ```bash
  docker run \
    --mount type=bind,source=/data/app,target=/app \
    nginx
  ```

  **Purpose**
  - Mount host directory
  - No Docker data abstraction


- **Read-Only Bind Mount**

  ```bash
  docker run \
    --mount type=bind,source=/etc/nginx,target=/etc/nginx,readonly \
    nginx
  ```


- **Bind Mount with Propagation**

  ```bash
  docker run \
    --mount type=bind,source=/mnt,target=/mnt,bind-propagation=rshared \
    mycontainer
  ```

  **Used for**
  - Docker-in-Docker
  - Nested mount
---

### 1.3 tmpfs Mount Using `--mount`

- **Basic tmpfs**

  ```bash
  docker run \
    --mount type=tmpfs,target=/run \
    nginx
  ```


- **tmpfs with Size Limit**

  ```bash
  docker run \
    --mount type=tmpfs,target=/cache,tmpfs-size=64m \
    myapp
  ```

  **Purpose**
  - Sensitive data
  - High-speed temporary storage

---

## 2. `-v / --volume` Option (Legacy, Short Syntax)

**General Syntax**

```bash
docker run -v <source>:<target>[:options] IMAGE
```

---

### 2.1 Using `-v` with Named Volumes

- **Auto Created Named Volume**

  ```bash
  docker run \
    -v db_data:/var/lib/mysql \
    mysql
  ```

  _**Output:** Volume auto-created if missing._


- **Read-Only Named Volume**

  ```bash
  docker run \
    -v app_config:/etc/app:ro \
    myapp
  ```

---

### 2.2 Bind Mount Using `-v`

- **Basic Bind Mount**
  ```bash
  docker run -v /data/app:/app nginx
  ```

- **Bind Mount Read-Only**

  ```bash
  docker run -v /etc/nginx:/etc/nginx:ro nginx
  ```

---

### 2.3 tmpfs with `-v`

- `-v` option is not supported here:
  ```bash
  docker run --tmpfs /run nginx
  ```

  _**Note:** `-v` is **not worked** for tmpfs, Use `--mount` instead._

---

## Best Practices

- Use `--mount` in production
- Pre-create volumes
- Never hard-code host paths unless needed
- Use read-only mounts where possible
- Separate storage creation from container run

---

## One-Line Exam Summary

- `--mount` is explicit, safer, and production-ready; 
- `-v` is shorthand and legacy but still widely used.

---


