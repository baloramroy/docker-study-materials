`docker network inspect`
# Docker Network Inspect

## Description

Returns **detailed information** about one or more Docker networks. By default, output is rendered in **JSON format**. Useful for **debugging**, **validation**, and **deep network analysis**.

**Usage**
```
docker network inspect [OPTIONS] NETWORK [NETWORK...]
```

---

## 1. Inspect a Network

### 1.1 Inspect detailed information of a network

```bash
docker network inspect my_network
```

_**Output Includes:**_
- _Driver_
- _Subnet and gateway_
- _IPAM configuration_
- _Connected containers and IPs_
- _Labels and driver options_

_**Purpose:** Deep troubleshooting and network analysis_


### 1.2 Inspect more than one network at once
  
```bash
docker network inspect bridge host
```

_**Output:** • JSON details for both networks_\
_**Purpose:** • Compare configurations between networks_
  
  
  
### 1.3 Inspect using network ID instead of name
```bash
docker network inspect $(docker network ls -q | head -n 1)
```
_**Output:** • JSON details of the first network in the list_\
_**Purpose:** • Useful in scripts and automation_

---

## 2. Filter Output Using `--format`

### 2.1 Show only subnet and IPAM configuration
  
```bash
docker network inspect --format '{{json .IPAM.Config}}' my_network
```

_**Output:**
• Subnet
• Gateway
• IP range_
  
  
  
### 2.2 Show only connected containers
  
```bash
docker network inspect --format '{{json .Containers}}' my_network
```

_**Output:**
• Container IDs
• Assigned IP addresses_



### 2.3 Show only the network driver
  
```bash
docker network inspect --format '{{.Driver}}' my_network
```

_**Output:** Driver name (bridge, overlay, macvlan, etc.)_

---

## 3. Inspect and Pretty‑Print Output

### 3.1 Pretty‑print full JSON using `jq`
  
```bash
docker network inspect my_network | jq
```

_**Output:** Human‑readable formatted JSON_
  
  
### 3.2 Pretty‑print a specific field
  
```bash
docker network inspect my_network | jq '.[0].Containers'
```

_**Output:** Connected containers in readable format_

---

## 4. Inspect Built‑in and Special Networks

### 4.1 Inspect the default bridge network
  
```bash
docker network inspect bridge
```

_**Output:**_
- _Linux bridge (`docker0`)_
- _Connected containers_
- _IP range and options_
  
  
  
### 4.2 Inspect the host network

```bash
docker network inspect host
```

_**Output:** • Host‑level networking details_\
_**Note:** • Containers share the host network namespace_
  
  
### 4.3 Inspect the none network

```bash
docker network inspect none
```

_**Output:** • Fully isolated network_


### 4.4 Inspect a Swarm overlay network

```bash
docker network inspect my_overlay_net
```

_**Output Includes:**_
- _Swarm scope_
- _Peers (nodes)_
- _Subnets and VXLAN details_

_**Requirement:** Docker Swarm mode must be enabled_

---

## 5. Inspect With Container‑Based Filtering (Indirect)

### 5.1 Find networks a specific container is connected to

```bash
docker network inspect \
  $(docker container inspect -f '{{json .NetworkSettings.Networks}}' webapp)
```

_**Output:** • All networks attached to container `webapp`_\
_**Purpose:** • Identify container‑to‑network relationships_

---

## 6. Summary

- `inspect` = **truth source** for Docker networking
- Default output = JSON
- `--format` = extract exact fields
- `jq` = human‑readable output
- Works with **names or IDs**
- Essential for debugging network issues

---
