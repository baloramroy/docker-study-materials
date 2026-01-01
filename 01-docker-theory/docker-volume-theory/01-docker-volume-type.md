To keep data outside the container (safe and persistent), Docker provides **three** types of mounts:

## Docker Storage Mount Options

Docker supports **three mount types**:

1. **Volume** – Docker-managed storage
2. **Bind mount** – Host directory mount
3. **tmpfs** – Memory-only storage

---

### 1. Volume

**What is a Volume?**

A **volume** is storage that:
- Is **managed by Docker**
- Lives under **Docker’s data** directory
- Is **independent of containers**

Docker decides:
- Where it is stored
- How it is mounted


**Simple Explanation**

Docker gives you a **storage box**, names it, and manages it for you. You only care about:
- Volume name
- Mount point inside container


**Where It Lives (Host)**

```text
/var/lib/docker/volumes/<volume_name>/_data
```

>You normally **don’t touch this path manually**.

**Example**

```bash
docker volume create db_data

docker run \
  --mount type=volume,source=db_data,target=/var/lib/mysql \
  mysql
```


**When to Use Volumes**

- Databases
- Application data
- Production workloads
- Multi-container sharing


**Pros / Cons**

Pros:
- Portable
- Safe
- Docker-managed
- Easy backup

Cons:
- Less control over exact host path


**Linux Analogy**

> Like letting **LVM** manage disks instead of mounting raw directories.

---

### 2. Bind Mount (Host Path Mount)

**What is a Bind Mount?**

A **bind mount**:
- Directly maps a **host directory** into a container
- Docker does **not manage** it
- Container sees the host filesystem **as-is**

**Simple Explanation**

> “Take this **exact folder** from my host and show it inside the container.”

**Example**

```bash
docker run \
  --mount type=bind,source=/data/app,target=/app \
  nginx
```

**What Happens Internally**

Linux does:

```bash
mount --bind /data/app /container/path
```

**When to Use Bind Mounts**
- Development
- Configuration files
- Logs
- Local testing

**Pros / Cons**
Pros:
- Full control
- Immediate file changes
- Simple

Cons:
- Not portable
- Host-dependent
- Security risks


**Linux Analogy**
> Same as `mount --bind` in Linux.

---

### 3. tmpfs (Memory-Only Storage)

**What is tmpfs?**

A **tmpfs mount**:
- Lives **only in RAM**
- Never touches disk
- Deleted when container stops

**Simple Explanation**
> “Create a **temporary scratchpad in memory**.”

**Example**

```bash
docker run --mount type=tmpfs,target=/run nginx
```

**tmpfs with Size Limit**

```bash
docker run --mount type=tmpfs,target=/cache,tmpfs-size=64m myapp
```

**When to Use tmpfs**
- Secrets
- Session data
- Temporary cache
- High-speed I/O

**Pros / Cons**

Pros
- Fast
- Secure
- Auto-cleaned

Cons
- Data loss on stop
- Uses RAM


**Linux Analogy**
> Same as mounting `tmpfs` at `/run` or `/dev/shm`.

---

## 4. One-Line Memory Trick

```
Volume   → Docker owns storage
Bind     → You own storage
tmpfs    → Memory only
```

---

## 5. Real-World Examples

**Database**\
*Volume*

**Config files**\
*Bind mount (read-only)*

**Passwords / tokens**\
*tmpfs*

---

