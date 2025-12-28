
---

# Understanding `docker volume create` (Advanced Options)

## 1. Big Picture (Very Important)

When you run:

```bash
docker volume create --driver local --opt ...
```

You are basically telling Docker:

> “Docker, **use Linux mount mechanisms** to attach some storage
> and manage it **as a Docker volume**.”

_**Key truth:**\
Docker volumes are **not magic storage** — they are wrappers around **Linux mounts**._

---

## 2. What is `--driver local`?

**Meaning**

```bash
--driver local
```

- Uses Docker’s **built-in volume driver**
- Internally relies on **Linux mount system calls**
- No external plugin needed

**In simple words**

> “Use the host machine’s local storage and mounting capabilities.”

**Real Linux analogy**

Same as running:

```bash
mount ...
```

on the **host**, but **Docker does it for you**.

---

## 3. What is `--opt`?

`--opt` means:

> “Pass **mount options** to the volume driver.”

Docker forwards these options **directly to the Linux mount command**.

---

## 4. Understanding `type`

**Example**

```bash
--opt type=none
```

or

```bash
--opt type=ext4
```

or

```bash
--opt type=nfs
```

**Meaning:**

`type` = **filesystem type**

**Linux equivalent:**

```bash
mount -t <type> ...
```

**Common values**

| type   | Meaning                              |
| ------ | ------------------------------------ |
| `none` | No filesystem (used for bind mounts) |
| `ext4` | Linux filesystem                     |
| `xfs`  | High-performance Linux filesystem    |
| `nfs`  | Network File System                  |

---

## 5. Understanding `device`

**Example**

```bash
--opt device=/data/app
```

```bash
--opt device=/dev/sdb1
```

```bash
--opt device=:/exports/docker
```

**Meaning**

`device` = **what you want to mount**

**Linux equivalent**

```bash
mount <device> <mount_point>
```

**Examples mapped**

| device value       | Meaning         |
| ------------------ | --------------- |
| `/data/app`        | Host directory  |
| `/dev/sdb1`        | Block device    |
| `:/exports/docker` | NFS export path |

---

## 6. Understanding `o` (options)

**Example**

```bash
--opt o=bind
```

```bash
--opt o=addr=192.168.1.10,rw
```

**Meaning**

`o` = **mount options**

**Linux equivalent**

```bash
mount -o <options>
```

**Common options**

| Option    | Purpose                  |
| --------- | ------------------------ |
| `bind`    | Bind mount a directory   |
| `rw`      | Read-write               |
| `ro`      | Read-only                |
| `addr=`   | NFS server address       |
| `noatime` | Performance optimization |

---

## 7. Now Let’s Decode Each Example Completely

### Example 1: Bind Mount Volume

```bash
docker volume create \
  --driver local \
  --opt type=none \
  --opt device=/data/app \
  --opt o=bind \
  app_bind_volume
```

**What Docker does internally**

Equivalent Linux command:

```bash
mount --bind /data/app /var/lib/docker/volumes/app_bind_volume/_data
```

**Explanation**

| Option             | Meaning                 |
| ------------------ | ----------------------- |
| `driver=local`     | Use local Linux storage |
| `type=none`        | No filesystem           |
| `device=/data/app` | Source directory        |
| `o=bind`           | Bind mount              |

**When to use**

- Map an **existing host directory**
- Logs, app data, shared configs

---

### Example 2: Block Device Volume

```bash
docker volume create \
  --driver local \
  --opt type=ext4 \
  --opt device=/dev/sdb1 \
  db_storage
```

**Internal Linux command**

```bash
mount -t ext4 /dev/sdb1 /var/lib/docker/volumes/db_storage/_data
```

**Explanation**

| Option             | Meaning                  |
| ------------------ | ------------------------ |
| `type=ext4`        | Filesystem               |
| `device=/dev/sdb1` | Dedicated disk/partition |

**When to use**

- Databases
- High I/O workloads
- Storage isolation

---

### Example 3: NFS Volume

```bash
docker volume create \
  --driver local \
  --opt type=nfs \
  --opt o=addr=192.168.1.10,rw \
  --opt device=:/exports/docker \
  nfs_volume
```

**Internal Linux command**

```bash
mount -t nfs 192.168.1.10:/exports/docker \
  /var/lib/docker/volumes/nfs_volume/_data
```

**Explanation**

| Option     | Meaning            |
| ---------- | ------------------ |
| `type=nfs` | Network filesystem |
| `addr=`    | NFS server         |
| `device=`  | Export path        |
| `rw`       | Read-write         |

### When to use

- Multi-host Docker environments
- Shared storage
- Legacy infrastructure

---

## One Golden Memory Trick

Remember this mapping **forever**:

```
Docker volume options  →  Linux mount command
--------------------------------------------
driver                →  mount utility
type                  →  -t
device                →  source
o                     →  -o
```

If you understand `mount`, you understand **Docker volumes deeply**.

---

## One-Line Summary

- Use `docker volume create` when storage must be **reusable, controlled, documented, and production-safe**.

- Use **--mount** options directly only for **temporary or simple cases**.

