# `docker network create`

## Description
Creates a **new** network. The DRIVER accepts **bridge or overlay** which are the **built-in network drivers**. If you have installed a **third party** or your own custom network driver you can specify that DRIVER here also. If you don't specify the **--driver** option, the command automatically creates a **bridge network** for you.
- Usage
  ```
  docker network create [OPTIONS] NETWORK
  ```

---

## Create a Bridge Network

- **Create a standard user-defined bridge network**
  ```bash
  docker network create my_bridge
  ```
  
  **Output:** Creates a custom bridge network.\
  **Note:** if you do not mention the driver, Docker always creates a bridge network by default.


- **Create a bridge network with custom IP range**
  ```bash
  docker network create --subnet=192.168.50.0/24 --gateway=192.168.50.1 my_bridge_net
  ```
  **Output:** Creates network with a custom subnet and gateway.\
  **Purpose:** Required in production when IP management is important.

- **Create network where containers get static IPs**
  ```bash
  docker network create --subnet=10.10.1.0/24 --ip-range=10.10.1.10/28 my_fixed_ip_net
  ```
  **Output:** Allows specific IP address range for containers.These are the IPs Docker will auto-assign to containers unless you manually specify an IP.

- **You can also manually assign an IP from anywhere in the subnet created earlier:**
  
  - From the allowed IP pool:
    ```
    docker run --network my_fixed_ip_net --ip 10.10.1.14 nginx
    ```
  
  - Or even outside the pool but inside the subnet:
    ```
    docker run --network my_fixed_ip_net --ip 10.10.1.50 nginx
    ```
  **Note:** Both are valid because the IP is still inside the subnet 10.10.1.0/24.

---

## Create a Driver-Specific Network

- **Use the bridge driver**
  ```bash
  docker network create -d bridge web_net
  ```
  **Output:** Bridge network created explicitly with driver.

- **Use the overlay driver (Swarm)**
  ```bash
  docker network create -d overlay web_overlay
  ```
  **Output:** Creates overlay network for multi-host connectivity.

- **Use the macvlan driver**
  ```bash
  docker network create -d macvlan --subnet=192.168.100.0/24 --gateway=192.168.100.1 -o parent=eth0 macvlan_net
  ```
  **Output:** Creates network where containers get real LAN IPs.

---

## Create an Internal-Only Network
- **Internal network (no outside access)**
  ```bash
  docker network create --internal internal_net
  ```
  **Output:** Containers cannot reach external networks.\
  **Purpose:** Security hardening (backend, DB-only networks).

---

## Create a Network with Custom Options

- **Set bridge isolation (per-container isolation)**
  ```bash
  docker network create --opt com.docker.network.bridge.enable_icc=false isolated_net
  ```
  **Output:** Blocks container-to-container communication.\
  **Purpose:** Security requirement in many production setups.

- **Set custom bridge name**
  ```bash
  docker network create --opt com.docker.network.bridge.name=br_custom custom_bridge
  ```
  **Output:** Creates Linux bridge named br_custom.

---

## Create a Network with Labels

```bash
docker network create --label env=prod --label team=devops prod_net
```
**Output:** Network gets metadata labels.\
**Purpose:** Useful for automation, CI/CD, inventory.

---  
