`--mount` vs `-v / --volume`
# Docker Volume Mounting Guide

## 1. Big Picture (Must Understand First)

Docker supports **three mount types**:

1. **Volume** (Docker-managed)
2. **Bind mount** (Host-path based)
3. **tmpfs** (Memory-only)

Both `--mount` and `-v` are **just syntaxes** to attach these mounts to a container.

---

## 2. `--mount` Option (Recommended, Explicit)

### 2.1 General Syntax

```bash
docker run \
  --mount type=<mount_type>,source=<src>,target=<dst>[,options] \
  IMAGE
```

### Key Fields

| Field     | Meaning                         |
| --------- | ------------------------------- |
| `type`    | volume / bind / tmpfs           |
| `source`  | Volume name or host path        |
| `target`  | Container path                  |
| `options` | ro, rw, propagation, tmpfs-size |

---

## 3. Mounting a Docker Volume Using `--mount`

### 3.1 Named Volume (Most Common)

```bash
docker run -d \
  --mount type=volume,source=db_data,target=/var/lib/mysql \
  mysql
```

**Purpose**
- Persistent storage
- Docker manages volume lifecycle

**Output:** This will mount the **app_config** volume readonly at `/etc/app` in the container.



### 3.2 Read-Only Named Volume

```bash
docker run \
  --mount type=volume,source=app_config,target=/etc/app,readonly \
  myapp
```

**Output:** This will mount the **app_config** volume readonly at `/etc/app` in the container.



### 3.3 Auto-Create Volume

```bash
docker run \
  --mount type=volume,source=new_volume,target=/data \
  alpine
```

• Docker creates `new_volume` automatically
• Uses default volume driver

---

## 4. Bind Mount Using `--mount`

### 4.1 Basic Bind Mount

```bash
docker run \
  --mount type=bind,source=/data/app,target=/app \
  nginx
```

**Purpose**
- Mount host directory
- No Docker data abstraction

---

### 4.2 Read-Only Bind Mount

```bash
docker run \
  --mount type=bind,source=/etc/nginx,target=/etc/nginx,readonly \
  nginx
```

---

### 4.3 Bind Mount with Propagation

```bash
docker run \
  --mount type=bind,source=/mnt,target=/mnt,bind-propagation=rshared \
  mycontainer
```

**Used for**
- Docker-in-Docker
- Nested mount
---

## 5. tmpfs Mount Using `--mount`

### 5.1 Basic tmpfs

```bash
docker run \
  --mount type=tmpfs,target=/run \
  nginx
```


### 5.2 tmpfs with Size Limit

```bash
docker run \
  --mount type=tmpfs,target=/cache,tmpfs-size=64m \
  myapp
```

**Purpose**
- Sensitive data
- High-speed temporary storage

---

## 6. `-v / --volume` Option (Legacy, Short Syntax)

### 6.1 General Syntax

```bash
docker run -v <source>:<target>[:options] IMAGE
```

---

## 7. Using `-v` with Named Volumes

```bash
docker run \
  -v db_data:/var/lib/mysql \
  mysql
```

• Volume auto-created if missing

---

### 7.1 Read-Only Volume

```bash
docker run \
  -v app_config:/etc/app:ro \
  myapp
```

---

## 8. Bind Mount Using `-v`

```bash
docker run \
  -v /data/app:/app \
  nginx
```

---

### 8.1 Bind Mount Read-Only

```bash
docker run \
  -v /etc/nginx:/etc/nginx:ro \
  nginx
```

---

## 9. tmpfs with `-v` (Limited Support)

```bash
docker run \
  -v tmpfs:/run \
  --tmpfs /run \
  nginx
```

⚠️ `-v` is **not recommended** for tmpfs
Use `--mount` instead.

---

## 10. Key Differences: `--mount` vs `-v`

| Feature          | `--mount`   | `-v`    |
| ---------------- | ----------- | ------- |
| Readability      | Very high   | Low     |
| Explicit fields  | Yes         | No      |
| Error prevention | High        | Low     |
| tmpfs support    | Full        | Limited |
| Production use   | Recommended | Legacy  |

---

## 11. Common Mistakes (Very Important)

### Mistake 1: Path Does Not Exist

```bash
-v /data/app:/app
```

• Docker auto-creates `/data/app`
• Leads to silent bugs

---

### Mistake 2: Wrong Source Type

```bash
--mount type=volume,source=/data,target=/app
```

❌ `/data` is a path, not volume name

---

### Mistake 3: Mixing Concepts

Bind mount when volume needed → portability issues

---

## 12. Best Practices (Production)

1. Use `--mount` in production
2. Pre-create volumes
3. Never hard-code host paths unless needed
4. Use read-only mounts where possible
5. Separate storage creation from container run

---

## 13. Mental Model (Remember This)

```
-v        → quick & dirty
--mount   → explicit & safe
```

If you understand **Linux mount**, `--mount` will feel natural.

---

## 14. One-Line Exam Summary

> **`--mount` is explicit, safer, and production-ready; `-v` is shorthand and legacy but still widely used.**

---

If you want next:

1. `--mount` **field-by-field deep dive**
2. Real **production troubleshooting**
3. Docker vs Kubernetes volume mapping
4. RHCE / CKA exam notes format

Just tell me which one.
