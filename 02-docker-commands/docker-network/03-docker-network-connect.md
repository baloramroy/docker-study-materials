`docker network connect`
# Docker Network Connect
## 1. Description

* Connects an **container (running or stopped)** to an **existing Docker network**
* After connection, containers can:
  * Communicate with other containers in the same network
  * Resolve each other using container name or alias

---

## 2. Command Syntax

```bash
docker network connect [OPTIONS] NETWORK CONTAINER
```

* `NETWORK` → Target Docker network
* `CONTAINER` → Container name or container ID

## 3. Commonly Used Options

- **`--alias`** - Adds extra DNS names (aliases) for the container within that network, useful for services like `db`, `redis`, or `mysql`.  
- **`--ip`** - Assigns a static IPv4 address to the container in the network. Requires the network to have a custom subnet.  
- **`--ip6`** - Assigns a static IPv6 address to the container in the network.  
- **`--gw-priority`** - Controls which network provides the default gateway. Higher values give higher priority.
- **`--link`** - Creates a legacy link to another container with an alias (deprecated; use user-defined networks + DNS instead).
- **`--link-local-ip`** - Adds a link-local IP address, mostly for advanced networking scenarios. 
- **`--driver-opt`** - Passes custom driver options (e.g., `sysctls`) to the network interface. Only supports `net.ipv4.*` and `net.ipv6.*` configurations.


---

## 4. Connect Container to a Network

### 4.1 Connect a container (Running or Stopped) to a network

```bash
docker network connect my_network mycontainer
```

**Output:** Container **mycontainer** becomes connected to **my_network**.\
**Purpose:** Enable communication with other containers on that network.

### 4.2 Connect Container using Network ID

```bash
docker network connect 8f3c1d2a mycontainer
```

**Output:** Container connects to the specified network by **ID**.\

### 4.3 Connect container at startup

```bash
docker run -itd --network=my_network busybox
```
----

## 5. Connect Container to Multiple Networks

### 5.1 Crete the Network First

- Create **frontend** network
    ```bash
    docker network create frontend-net
    ```
- Create **backend** network  
    ```bash
    docker network create backend-net
    ```

### 5.2 Start a container initially connected to frontend network
```bash
docker run -d --name webapp --network frontend-net mycontainer
```
**Output:** Container now has **connected to frontend network**.


### 5.3 Connect container to an additional backend network
```bash
docker network connect backend_net mycontainer
```

**Output:** Container now has **multiple network interfaces**.\
**Purpose:** Frontend ↔ backend separation. 

---

## 6. Add Network Alias

### 6.1 Connect with network-scoped alias

```bash
docker network connect --alias db my_network mysql01
```

**Output:** Container reachable as **db** inside the network.\
**Purpose:** Service discovery without hardcoding container names.

### 6.2 Multiple aliases

```bash
docker network connect --alias api --alias backend my_network app01
```

**Output:** Container accessible via **api** and **backend** aliases.

---

## 7. Assign a Static IP Address

### 7.1 Connect container with a fixed IPv4 address

```bash
docker network connect --ip 192.168.10.50 my_network mycontainer
```

**Output:** Container receives **static IP** 192.168.10.50.\
**Purpose:** Legacy apps, monitoring tools, IP-based access rules.


### 7.2 Connect container with IPv6 address

```bash
docker network connect --ip6 2001:db8:1::50 my_network mycontainer
```

**Output:** Container gets **static IPv6** address.

---

## 8. Define Gateway Priority

### 8.1 `--gw-priority`

* Controls which network provides the **default gateway**
* Higher value = higher priority
* Can be **positive or negative**

### 8.2 Connect container with gateway priority

```bash
docker network connect --gw-priority 1 my_network container1
```

**Output:** Container uses **my_network** as its **default gateway**.

**Purpose:**\
When a container is connected to **multiple networks**, this option decides **which network handles outbound traffic (0.0.0.0/0)**.


### 8.3 Multiple networks with different priorities

```bash
docker network connect --gw-priority 10 frontend_net container1
docker network connect --gw-priority 1 backend_net container1
```

**Output:**
Default gateway is set via **frontend_net**.

**Purpose:**
Explicitly control routing when a container spans **frontend + backend** networks.

---

## 9. Legacy Container Linking

### 9.1 Link container with alias (legacy)

```bash
docker network connect --link container1:c1 my_network container2
```

**Output:** `container2` can reach `container1` using hostname **c1**.\
**Purpose:** Old-style container communication before Docker DNS existed.

> ⚠️ Avoid using `--link` in modern setups.

---

## 10. Connect with Driver Options

### 10.1 Pass driver-specific options

```bash
docker network connect \
  --driver-opt com.docker.network.endpoint.exposedports=80 \
  my_network mycontainer
```

**Output:** Applies **driver options** to the network endpoint.\
**Purpose:** Advanced networking configurations.

---

## 11. Important Notes

* Static IP (`--ip`) works only on **user-defined bridge or overlay networks**
* `--link` is **deprecated**
* Some `sysctl` values may be:
  * Blocked by Docker
  * Restricted by the network driver
* A container may have:
  * Multiple IPs (one per network)
  * Multiple default gateways (controlled by `--gw-priority`)

---
