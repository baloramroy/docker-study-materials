
# Linux Namespaces and CGroups

### First, Understand the Core Idea

Before namespaces and cgroups, Linux had only **processes**.
#
**All processes could:**
* See each other
* Compete freely for CPU and memory
* Share the same filesystem view
* Share the same network stack

This was **not safe**, **not isolated**, and **not scalable**.

#
**Linux solved this with two kernel features**:

1. **Namespaces** → *Isolation (What a process only can see)*
2. **cgroups** → *Resource control (What a process only  can use)*

Containers = **Processes + Namespaces + cgroups**\
Nothing more.

>The Real Foundation of Containers

---

## 1. Namespace

### What is a Namespace?

**Definition**

A **namespace** is a Linux kernel feature that **isolates a global system resource** so that a process thinks it has its **own private version** of that resource.

> Same kernel, but **different views** of reality.


### Types of Linux Namespaces

Linux supports **8 namespaces**. Containers use almost all of them.

| Namespace | What it isolates            |
| --------- | --------------------------- |
| PID       | Process IDs                 |
| NET       | Network stack               |
| MNT       | Mount points                |
| UTS       | Hostname & domain           |
| IPC       | Inter-process communication |
| USER      | User & group IDs            |
| CGROUP    | cgroup hierarchy            |
| TIME      | System time (newer kernels) |

We’ll explore them **one by one**.

---

### PID Namespace (Process Isolation)

**A PID namespace** is a Linux kernel feature that **isolates process IDs** so that a **group of processes** sees only **its own processes** and has **its own PID numbering.**

**Problem Without PID Namespace**

* Every process can see **all system processes**
* `ps aux` shows everything
* Killing PID 1 affects the whole system

**What PID Namespace Does**

* Each namespace has its own **process tree**
* Every namespace has its own `PID 1`
* `ps`, `top`, `/proc` show only **local processes**
* Processes **outside** the namespace cannot see **inside**


**Result of this isolation**

* Each container has its **own process tree**
* `ps`, `top`, and `/proc` show **only container processes**
* PID values are **reused safely** across containers
* Killing PID 1 affects **only that container**
* Host processes remain **fully protected and invisible**


**Key Rules**

- **PID namespaces are hierarchical**\
  *Parent can see child processes*\
  *Child cannot see parent processes*

- **PID 1 is special**\
  *Responsible for Reaping zombie processes*\
  *Responsible for Signal handling*\
  *If PID 1 dies → namespace dies*

- **Processes join PID namespace at creation**\
  *Cannot be moved later*\
  *Requires `clone()` or `unshare`*

- **`/proc` is PID-namespace aware**\
  *`/proc` only shows processes in that namespace*


**Real Container Example**

- Inside a container:
   ```
   PID 1 → nginx
   PID 2 → worker
   ```

- On the host:
   ```
   PID 23456 → nginx
   PID 23457 → worker
   ```

  >They are the **same processes**, but their **ids** id different as they are in **isolated views**.

**Command to See It**

- Check namespaces of a process:

    ```bash
    ls -l /proc/<pid>/ns/
    ```
- See PID mapping:

  ```bash
  ps -ef
  ```

>**PID namespace** = “Your own private process universe”

---

### NET Namespace (Network Isolation)

**A NET namespace** is a Linux kernel feature that **isolates the network stack** so that a group of processes has its **own independent networking environment.**

**Problem Without NET Namespace**
* All processes share the same network interfaces
* IP addresses are global
* Port numbers are global (only one process can bind to port 80)
* Firewall rules affect the whole system


**What NET Namespace Does**

With NET namespace, each namespace gets its **own complete network stack**, including:
* Network interfaces
* IP addresses
* Routing table
* Firewall rules
* Ports

**Result of this isolation**

* Each container has its **own IP address**
* Multiple containers can bind to **the same port** (e.g., 80)
* Network traffic is **fully isolated**
* Host network remains **unaffected**
* Clean **separation** between containers

**Key Rules**

- **Network namespaces are fully isolated**\
  *No shared interfaces by default*

- **Each NET namespace starts with only `lo`**\
  *External access requires virtual interfaces (veth)*

- **Port conflicts do not exist across namespaces**\
  *Same port number can be reused safely*

- **NET namespace ≠ DNS isolation**\
  *DNS comes from `/etc/resolv.conf` (MNT namespace)*

- **Bridges connect namespaces**\
  *Docker uses Linux bridges (e.g., `docker0`)*

**Real Container Example**

- Inside container:

    ```
    eth0 → 172.17.0.2
    ```

- Host:

    ```
    eth0 → 192.168.1.10
    ```

  >They cannot see each other unless explicitly connected.

**Command to See It**

- Check network namespace:

  ```bash
  ls -l /proc/self/ns/net
  ```

>**NET namespace** = “Your own private network universe”

---

### MNT Namespace (Mount / Filesystem Isolation)

A **MNT (Mount) namespace** is a Linux kernel feature that **isolates filesystem mount points** so that a group of processes has **its own view of the filesystem**.

**Problem Without MNT Namespace**

* All processes share the same mount table
* `mount` or `umount` affects the entire system
* Any process can see **all mounted filesystems**
* No safe way to give a process its own `/` root

**What MNT Namespace Does**

* Each namespace has its **own mount table**
* Mount and unmount affect **only that namespace**
* Processes can have a **different root filesystem**
* Host filesystem can be **hidden from containers**

**Result**

* Containers see **their own `/` (root filesystem)**
* Mount changes affect **only that container**
* Host filesystem remains **hidden and safe**
* Multiple containers can mount **different filesystems independently**

**Key Rules**

- **MNT namespace isolates mount points, not files**\
  *Same underlying filesystem can be mounted in multiple places*

- **Each namespace starts as a copy of the parent**\
  *Isolation happens after namespace creation*

- **Mount propagation matters**\
  *private → not shared*\
  *shared → visible across namespaces*\
  *slave → one-way sharing*

- **Works together with `pivot_root`**\
  *Containers change `/` safely using MNT namespace*

- **chroot ≠ container**\
  *chroot alone is insecure*\
  *chroot + MNT namespace = container filesystem isolation*

**Real Container Example**

- Inside container:

  ```
  / → container rootfs
  /proc → container proc
  /etc → container config
  ```

- Inside Host:

  ```
  / → host rootfs
  ```

  >They are **not the same view**.


**Command to See It**

- Check mount namespace:

  ```bash
  ls -l /proc/self/ns/mnt
  ```

- Show mounts:

  ```bash
  mount
  findmnt
  ```

>**MNT namespace** = “Your own private filesystem view”

---

### UTS Namespace (Hostname Isolation)

A **UTS (UNIX Time-Sharing) namespace** is a Linux kernel feature that **isolates the system identity** of a process.


**Problem Without UTS Namespace**

* All processes share the same hostname
* Changing hostname affects the entire system
* Containers cannot identify themselves uniquely
* Logs from different containers look confusing

**What UTS Namespace Does**

* Each namespace has its **own hostname**
* Each namespace has its **own domain name**
* Hostname changes are **local to the namespace**

**Result**

* Every container can have a **unique hostname**
* Changing hostname inside container does **not affect host**
* Cleaner logging and monitoring
* Easier container identification and debugging

**Key Rules**

- **UTS namespace only isolates identity**\
  *Does **not** isolate network, IP, or DNS*

- **Commands affected**\
*- `hostname`*\
*- `uname -n`*

- **DNS is not handled by UTS**\
  *DNS comes from `/etc/resolv.conf` (MNT namespace)*

- **Almost always used with NET namespace**\
  *Hostname alone is not useful without network isolation*

**Real Container Example**

- Inside container:

  ```
  hostname → web-01
  ```

- Host:

  ```
  hostname → prod-server
  ```

  Both coexist without conflict.


**Command to See It**

- Check UTS namespace:

  ```bash
  ls -l /proc/self/ns/uts
  ```

- Show hostname:

  ```bash
  hostname
  uname -n
  ```

>**UTS namespace** = “Your system’s name tag”

---

### IPC Namespace (Inter-Process Communication Isolation)

**What is IPC in Linux**

IPC **(Inter-Process Communication)** means how processes **talk to each other** without using **files or network**.

**Problem Without IPC Namespace**

* Processes can access shared memory segments
* Signals and message queues are globally visible
* Containers can interfere with each other

**What IPC Namespace Does**

IPC namespace **separates these communication mechanisms** so that:
* Processes in one container can talk **only among themselves**,
* and **cannot see or use** IPC objects created by other containers or the host.

Example: 

i. System V IPC:
* An **old but still used** IPC system in Linux. 
* Provides:
  - Shared memory
  - Semaphores
  - Message queues
* System V IPC objects are **visible only inside that container**

ii. POSIX Message Queues
* Modern **message-passing** system
* Used for **fast, structured communication** between processes
* Each container has its **own message queue space**

iii. Shared Memory
* Memory shared between processes
* Faster than files or sockets
* Shared memory exists **only inside that namespace**

**Result**

* Containers cannot see or access **IPC objects** of others
* Shared memory stays **inside container boundary**
* No **cross-container** IPC interference
* **Stronger** security and stability

**Key Rules**

- **IPC namespace does NOT isolate Unix domain sockets**\
  *Those are filesystem-based*\
  *Isolated by **MNT namespace + permissions***

- **IPC namespace is usually paired with PID namespace**\
  *IPC objects are tied to process ownership*

- **IPC objects have UID/GID ownership**\
  *USER namespace improves security*

- **Most containers use IPC namespace by default**\
  *Can be shared intentionally (e.g., Kubernetes Pods)*


## Real Container Example

- Inside container:

  ```
  Shared memory → app processes only
  Message queues → container only
  ```

- Host and other containers:

  ```
  No visibility
  ```

**Commands to See It**

- Check IPC namespace:

  ```bash
  ls -l /proc/self/ns/ipc
  ```

- List IPC objects:

  ```bash
  ipcs
  ```
>**IPC namespace** = “Private communication room for processes”

---

## USER Namespace (User & Permission Isolation)

A **USER namespace** is a Linux kernel feature that **isolates user and group IDs (UIDs/GIDs)** so that a process can have **different user identities inside and outside** the namespace.

**Problem Without USER Namespace**

* Root inside container = root on host
* Any container breakout becomes **full host compromise**
* Unprivileged users cannot safely run containers
* Poor security boundary

**What USER Namespace Does**

*  Allows **UID/GID mapping** between container and host
*  `root (UID 0)` inside container maps to **non-root UID** on host
*  Privileges are **limited automatically**

Example:

```
Inside container:  UID 0 (root)
On host:           UID 100000
```

Same process, different identities.

**Result**

* Root inside container has **limited host privileges**
* Even if container is compromised, host stays **secure**
* Enables **rootless containers** (Podman, Docker rootless)
* Limits damage if a container is compromised

**Key Rules**

- **USER namespace must be created first**\
  *Other namespaces depend on it for permissions*

- **UID/GID mappings are fixed at creation**\
  *Cannot be changed later*

- **Capabilities are filtered**\
  *Even container root lacks many kernel powers*

- **Mapping is defined in**

   * `/etc/subuid`
   * `/etc/subgid`

- **USER namespace ≠ authentication**\
  *It controls kernel permissions, not login users*

## Real Container Example

- Inside container:
  ```
  whoami → root
  ```

- On host:
  ```
  ps shows UID → 100000
  ```

Root inside container cannot:
- Load kernel modules
- Mount host filesystems
- Kill host processes

**Commands to See It**

- Check USER namespace:

  ```bash
  ls -l /proc/self/ns/user
  ```

- Check UID mapping:

  ```bash
  cat /proc/self/uid_map
  cat /proc/self/gid_map
  ```

>**USER namespace** = “Root without real power”

---

## CGROUP Namespace

A **cgroup namespace** is a Linux kernel feature that **isolates the view of control groups (cgroups)** so that a process sees **only its own cgroup hierarchy**.

**Problem Without cgroup namespace:**

- Processes can see the **host’s entire cgroup tree**
- Containers can discover host **resource layout**
- Monitoring tools inside containers show **confusing host data**


**What cgroup Namespace Does**

- Each namespace sees a **virtualized cgroup path**
- Processes see **only their own cgroup and children**
- Host cgroup hierarchy is **hidden**

Example:

```
Inside container:  /
Host reality:      /sys/fs/cgroup/docker/<container-id>/
```

**Result (What You Actually Gain)**

- Clean and simple **cgroup view** inside containers
- Container tools report **correct resource limits**
- Host resource structure remains **private**
- Better isolation for monitoring and debugging

**Key Rules**

- **cgroup namespace does NOT enforce limits**
  *It only hides the cgroup layout*\
  *Actual limits are enforced by **cgroup controllers***

- **Works with cgroup v2**
  *Designed mainly for unified hierarchy*

- **Usually enabled automatically**
  *Docker, Podman, Kubernetes use it by default*

- **Limit visibility, not control**
  *Limits come from CPU, memory, IO controllers*


## Real Container Example

- Inside container:

  ```
  cat /proc/self/cgroup
  → 0::/
  ```

- Host:

  ```
  /sys/fs/cgroup/docker/<container-id>/
  ```


**Commands to See It**

- Check cgroup namespace:

  ```bash
  ls -l /proc/self/ns/cgroup
  ```

- View cgroup info:

  ```bash
  cat /proc/self/cgroup
  ```

>**cgroup namespace** = “Blindfold for resource hierarchy”

---

## TIME Namespace

A **TIME namespace** is a Linux kernel feature that allows a process to see a **different system time offset** than the **host**.


**Linux has two kinds of time:**
- **Real time** → what you **see with date**
- **Internal time counters** → used by programs to measure **how long something runs**

>TIME namespace touches only #2, never #1.


**What TIME Namespace Does**

TIME namespace lets a process think:

>“The system started earlier/later than it really did.”

That’s it.
Nothing more.

So the process may think:
- The system has been running for 10 hours
- While the host has been running for 2 hours

This means:
* Uptime can look different than host
* Timers can be offset
* Time-based behavior can be tested

With TIME namespace, a process can have:

- A different **CLOCK_MONOTONIC**\  
  An internal clock that measures the time since the system was **booted**,
  **without** the time when the system was **suspended**.

- A different **CLOCK_BOOTTIME**\  
  An internal clock that measures the time since the system was booted,
  **including** the time spent in system **suspend**.


**What It Does NOT Do**

- **TIME namespace does not change real system time**
- It does not Change date
- It does not Affect other processes
- It does not Affect host time
- `date` usually shows the **host** time


**Why TIME Namespace Exists**

TIME namespace was created mainly for:

• Testing time-sensitive applications
• Simulating clock drift
• Debugging timeout-related bugs
• Running legacy software that depends on uptime

It is **not** required for normal containers.


**Result (What You Actually Gain)**

- A process can think it started “earlier” or “later”
- Uptime inside container can differ from host
- Time-based bugs can be reproduced safely


**Key Rules**

- **TIME namespace does NOT change real system time**\
  *Host clock stays unchanged*

- **Only monotonic clocks are affected**\
  *`date` usually shows host time*

- **Rarely used in production**\
  *Mostly for testing and research*

- **Requires modern kernel**\
  *Linux 5.6+*

---

# cgroups

## What Is a cgroup?

A **control group (cgroup)** is a Linux kernel feature that **limits, accounts, and prioritizes** `resources` for a **group of processes**.

Namespaces say:
* “What you can see”

cgroups say:
* “What you can use”


## What Resources cgroups Control

| Resource | Controller        |
| -------- | ----------------- |
| CPU      | cpu               |
| Memory   | memory            |
| Disk I/O | blkio             |
| Network  | net_cls, net_prio |
| PIDs     | pids              |
| Freezer  | freezer           |

---

## 14. CPU cgroups (How Much CPU)

### Controls

* CPU shares
* CPU quota
* CPU throttling

### Example

Limit container to **50% CPU**:

```
cpu.cfs_quota_us
cpu.cfs_period_us
```

---

## 15. Memory cgroups (OOM Control)

### Controls

* Maximum memory usage
* Swap usage
* OOM behavior

### Why This Matters

Without memory cgroups:

* One process can kill the host
  With memory cgroups:
* Only the container dies

---

## 16. Disk I/O cgroups

Controls:

* Read/write speed
* IOPS limits

Prevents:

* One container flooding disk

---

## 17. PIDs cgroup

Limits:

* Number of processes

Prevents:

* Fork bombs inside containers

---

## 18. cgroup v1 vs cgroup v2 (You Must Know)

### cgroup v1

* Multiple hierarchies
* Complex
* Legacy

### cgroup v2 (Modern)

* Single unified hierarchy
* Stronger security
* Used by:

  * systemd
  * Kubernetes
  * Podman

Check version:

```
mount | grep cgroup
```

---

## 19. How Docker Uses Namespaces & cgroups

Docker does **not invent anything**.

It does:

1. Create namespaces using `clone()`
2. Configure mounts
3. Apply cgroups limits
4. Start a process

That’s it.

---

## 20. Debugging Containers at Namespace Level

### Enter container namespaces:

```
nsenter --target <pid> --mount --pid --net --uts --ipc bash
```

### Inspect cgroups:

```
cat /proc/<pid>/cgroup
```

---

## 21. Final Mental Model (Very Important)

* Container ≠ VM
* Container = **Linux process**
* Namespaces = walls
* cgroups = resource meters
* Kernel = shared

---



