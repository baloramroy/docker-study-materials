# `docker network create`

## Description

Creates a **new Docker network**.

- Docker provides **built-in network drivers** such as **bridge**, **overlay**, **host**, **none**, and **macvlan**.
- You can also use **third‑party or custom drivers** if installed.
- If you do **not** specify `--driver`, Docker **defaults to a user‑defined bridge network**.

**Usage**

```
docker network create [OPTIONS] NETWORK
```

---

## 1. Create a Bridge Network

### 1.1 Create a standard user‑defined bridge network

```bash
docker network create my_bridge
```

**Result:** Creates a custom bridge network.\
**Note:** User‑defined bridge networks provide **automatic DNS**, **container name resolution**, and **better isolation** than the default `bridge`.


### 1.2 Create a bridge network with a custom subnet and gateway

```bash
docker network create \
  --subnet=192.168.50.0/24 \
  --gateway=192.168.50.1 \
  my_bridge_net
```

**Result:** Network created with a defined IP range.

**Purpose:** Required in production when **IP address planning** and **predictability** matter.


### 1.3 Create a bridge network with a restricted auto‑assignment IP range

```bash
docker network create \
  --subnet=10.10.1.0/24 \
  --ip-range=10.10.1.10/28 \
  my_fixed_ip_net
```

**Result:** Docker auto‑assigns container IPs **only from 10.10.1.10–10.10.1.25**.

**Important Concept:**
- `--subnet` → total available IP space
- `--ip-range` → pool Docker uses for **automatic** assignment


### 1.4 Manually assign container IP addresses

**From the allowed IP pool**

```bash
docker run --network my_fixed_ip_net --ip 10.10.1.14 nginx
```

**Outside the auto‑assign pool but inside the subnet**

```bash
docker run --network my_fixed_ip_net --ip 10.10.1.50 nginx
```

**Note:**
- Both are valid because the IP belongs to `10.10.1.0/24`.
- Docker does **not** block manual IPs inside the subnet.

---

## 2. Create a Driver‑Specific Network

### 2.1 Explicitly use the bridge driver

```bash
docker network create -d bridge web_net
```

**Result:** Bridge network created explicitly.


### 2.2 Use the overlay driver (Docker Swarm only)

```bash
docker network create -d overlay web_overlay
```

**Result:** Overlay network for **multi‑host container communication**.\
**Important:** Requires **Docker Swarm mode** (`docker swarm init`).


### 2.3 Create a macvlan network (containers get real LAN IPs)

```bash
docker network create -d macvlan \
  --subnet=192.168.100.0/24 \
  --gateway=192.168.100.1 \
  -o parent=eth0 \
  macvlan_net
```

**Result:** Containers appear as **physical devices** on the LAN.

**Use Cases:**
- Legacy applications
- Network appliances
- Direct LAN access

---

## 3. Create an Internal‑Only Network

### 3.1 Internal (isolated) network

```bash
docker network create --internal internal_net
```

**Result:** Containers cannot reach external networks.

**Purpose:**
- Backend services
- Databases
- Security hardening

---

## 4. Create a Network with Custom Options

### 4.1 Disable inter‑container communication (ICC)

```bash
docker network create \
  --opt com.docker.network.bridge.enable_icc=false \
  isolated_net
```

**Result:** Containers on the same network **cannot communicate directly**.

**Purpose:**
- High‑security environments
- Zero‑trust container design


### 4.2 Use a custom Linux bridge name

```bash
docker network create \
  --opt com.docker.network.bridge.name=br_custom \
  custom_bridge
```

**Result:** Creates a Linux bridge named `br_custom` on the host.

---

## 5. Create a Network with Labels

```bash
docker network create \
  --label env=prod \
  --label team=devops \
  prod_net
```

**Result:** Network includes metadata labels.

**Purpose:**
- Automation
- CI/CD pipelines
- Environment classification

---

## Summary
- **No driver specified** → bridge
- **Bridge** → single host
- **Overlay** → multi‑host (Swarm)
- **Macvlan** → real LAN IPs
- **Internal** → no external access
- **ip‑range** limits auto‑assignment only

---
