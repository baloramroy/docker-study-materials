`docker network disconnect`
# Docker Network Disconnect

## Description
Disconnects a container from a network. The container must be running to disconnect it from the network.

**Usage**

```
docker network disconnect [OPTIONS] NETWORK CONTAINER
```
---

## 1. Disconnect a Container from a Network

### 1.1 Disconnect a running container from a network

```bash
docker network disconnect my_network mycontainer
```

**Output:** Container **mycontainer** is removed from **my_network**.\
**Purpose:** Stop container communication with that network.


### 1.2 Disconnect Using Network ID

```bash
docker network disconnect 8f3c1d2a mycontainer
```

**Output:** Container is disconnected from the network by **ID**.\
**Purpose:** Useful in scripts and automation.


### 1.3 Force disconnect even if container is running

```bash
docker network disconnect --force my_network mycontainer
```

**Output:** Container is **immediately disconnected**.\
**Purpose:** Resolve stuck networking, conflicts, or misconfigurations.

---

## 2. Disconnect a Stopped Container

### 2.1 Disconnect a stopped container

```bash
docker network disconnect my_network mycontainer
```

**Output:** Network is removed from container config.\
**Purpose:** Clean up networking before restarting container.

---

### 3. Disconnect from One Network When Multiple Are Attached

### 3.1 Remove container from a specific network

```bash
docker network disconnect backend_net mycontainer
```

**Output:** Container remains running on **other networks**.

---





