To keep data outside the container (safe and persistent), Docker provides **three** types of mounts:

## 2. Docker Storage Mount Options

Docker supports **three mount types**:

1. **Volume** – Docker-managed storage
2. **Bind mount** – Host directory mount
3. **tmpfs** – Memory-only storage

---

## 1. Volume

1.1 What is a Volume?

A **volume** is storage that:
• Is **managed by Docker**
• Lives under Docker’s data directory
• Is **independent of containers**

Docker decides:
• Where it is stored
• How it is mounted


### 1.2 Simple Explanation

> “Docker gives you a **storage box**, names it, and manages it for you.”

You only care about:
• Volume name
• Mount point inside container


### 1.3 Where It Lives (Host)

```text
/var/lib/docker/volumes/<volume_name>/_data
```

You normally **don’t touch this path manually**.

---

### 1.4 Example

```bash
docker volume create db_data

docker run \
  --mount type=volume,source=db_data,target=/var/lib/mysql \
  mysql
```

---

### 1.5 When to Use Volumes

• Databases
• Application data
• Production workloads
• Multi-container sharing

---

### 1.6 Pros / Cons

**Pros**
• Portable
• Safe
• Docker-managed
• Easy backup

**Cons**
• Less control over exact host path

---

### 1.7 Linux Analogy

> Like letting **LVM** manage disks instead of mounting raw directories.

---

## 2. Bind Mount (Host Path Mount)

### 2.1 What is a Bind Mount?

A **bind mount**:
• Directly maps a **host directory** into a container
• Docker does **not manage** it
• Container sees the host filesystem **as-is**

---

### 2.2 Simple Explanation

> “Take this **exact folder** from my host and show it inside the container.”

---

### 2.3 Example

```bash
docker run \
  --mount type=bind,source=/data/app,target=/app \
  nginx
```

---

### 2.4 What Happens Internally

Linux does:

```bash
mount --bind /data/app /container/path
```

---

### 2.5 When to Use Bind Mounts

• Development
• Configuration files
• Logs
• Local testing

---

### 2.6 Pros / Cons

**Pros**
• Full control
• Immediate file changes
• Simple

**Cons**
• Not portable
• Host-dependent
• Security risks

---

### 2.7 Linux Analogy

> Same as `mount --bind` in Linux.

---

## 3. tmpfs (Memory-Only Storage)

### 3.1 What is tmpfs?

A **tmpfs mount**:
• Lives **only in RAM**
• Never touches disk
• Deleted when container stops

---

### 3.2 Simple Explanation

> “Create a **temporary scratchpad in memory**.”

---

### 3.3 Example

```bash
docker run \
  --mount type=tmpfs,target=/run \
  nginx
```

---

### 3.4 tmpfs with Size Limit

```bash
docker run \
  --mount type=tmpfs,target=/cache,tmpfs-size=64m \
  myapp
```

---

### 3.5 When to Use tmpfs

• Secrets
• Session data
• Temporary cache
• High-speed I/O

---

### 3.6 Pros / Cons

**Pros**
• Fast
• Secure
• Auto-cleaned

**Cons**
• Data loss on stop
• Uses RAM

---

### 3.7 Linux Analogy

> Same as mounting `tmpfs` at `/run` or `/dev/shm`.

---

## 4. Side-by-Side Comparison (Very Important)

| Feature           | Volume | Bind Mount | tmpfs          |
| ----------------- | ------ | ---------- | -------------- |
| Managed by Docker | Yes    | No         | Yes            |
| Persistent        | Yes    | Yes        | No             |
| Uses Host Path    | Hidden | Explicit   | No             |
| Performance       | Good   | Good       | Very fast      |
| Portable          | Yes    | No         | Yes            |
| Production Use    | Best   | Limited    | Specific cases |

---

## 5. One-Line Memory Trick (Golden)

```
Volume   → Docker owns storage
Bind     → You own storage
tmpfs    → Memory only
```

---

## 6. Real-World Examples

### Database

→ **Volume**

### Config files

→ **Bind mount (read-only)**

### Passwords / tokens

→ **tmpfs**

---

## 7. Exam-Ready Summary (RHCE / CKA Style)

• **Volume**: Docker-managed persistent storage
• **Bind mount**: Direct host directory mapping
• **tmpfs**: Non-persistent memory storage

---

If you want next:

1. **Common mistakes per mount type**
2. **Security implications**
3. **Docker vs Kubernetes volume mapping**
4. **Hands-on lab scenarios**

Just tell me which one.
