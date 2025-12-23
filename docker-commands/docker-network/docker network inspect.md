# `docker network inspect`

## Description

Returns **detailed information** about one or more Docker networks. By default, output is rendered in **JSON format**. Useful for **debugging**, **validation**, and **deep network analysis**.

- **Usage**

  ```
  docker network inspect [OPTIONS] NETWORK [NETWORK...]
  ```

---

## 1. Inspect a Network

- ### Inspect detailed information of a network

  ```bash
  docker network inspect my_network
  ```
  
  **Output Includes:**
  - Driver
  - Subnet and gateway
  - IPAM configuration
  - Connected containers and IPs
  - Labels and driver options
  
  **Purpose:**
  • Deep troubleshooting and network analysis



- ### Inspect more than one network at once
  
  ```bash
  docker network inspect bridge host
  ```
  
  **Output:**
  • JSON details for both networks\
  **Purpose:**
  • Compare configurations between networks
  
  
  
- ### Inspect using network ID instead of name
  ```bash
  docker network inspect $(docker network ls -q | head -n 1)
  ```
  **Output:** • JSON details of the first network in the list\
  **Purpose:** • Useful in scripts and automation

---

## 2. Filter Output Using `--format`

- ### Show only subnet and IPAM configuration
  
  ```bash
  docker network inspect \
    --format '{{json .IPAM.Config}}' \
    my_network
  ```
  
  **Output:**
  • Subnet
  • Gateway
  • IP range
  
  
  
- ### Show only connected containers
  
  ```bash
  docker network inspect \
    --format '{{json .Containers}}' \
    my_network
  ```
  
  **Output:**
  • Container IDs
  • Assigned IP addresses



- ### Show only the network driver
  
  ```bash
  docker network inspect \
    --format '{{.Driver}}' \
    my_network
  ```
  
  **Output:**
  • Driver name (bridge, overlay, macvlan, etc.)

---

## 3. Inspect and Pretty‑Print Output

- ### Pretty‑print full JSON using `jq`
  
  ```bash
  docker network inspect my_network | jq
  ```
  
  **Output:**
  • Human‑readable formatted JSON
  
  
  
- ### Pretty‑print a specific field
  
  ```bash
  docker network inspect my_network | jq '.[0].Containers'
  ```
  
  **Output:**
  • Connected containers in readable format

---

## 4. Inspect Built‑in and Special Networks

- ### Inspect the default bridge network
  
  ```bash
  docker network inspect bridge
  ```
  
  **Output:**
  • Linux bridge (`docker0`)
  • Connected containers
  • IP range and options
  
  
  
- ### Inspect the host network
  
  ```bash
  docker network inspect host
  ```
  
  **Output:**
  • Host‑level networking details\
  **Note:**
  • Containers share the host network namespace
  
  
- ### Inspect the none network
  
  ```bash
  docker network inspect none
  ```
  
  **Output:**
  • Fully isolated network


- ### 4.4 Inspect a Swarm overlay network
  
  ```bash
  docker network inspect my_overlay_net
  ```
  
  **Output Includes:**
  • Swarm scope
  • Peers (nodes)
  • Subnets and VXLAN details\
  **Requirement:**
  • Docker Swarm mode must be enabled

---

## 5. Inspect With Container‑Based Filtering (Indirect)

- ### Find networks a specific container is connected to
  
  ```bash
  docker network inspect \
    $(docker container inspect -f '{{json .NetworkSettings.Networks}}' webapp)
  ```
  
  **Output:**
  • All networks attached to container `webapp`\
  **Purpose:**
  • Identify container‑to‑network relationships

---

## 6. Summary

- `inspect` = **truth source** for Docker networking
- Default output = JSON
- `--format` = extract exact fields
- `jq` = human‑readable output
- Works with **names or IDs**
- Essential for debugging network issues

---
