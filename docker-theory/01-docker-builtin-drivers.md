# Docker Builtin Network Drivers Explanation

## Docker built in drivers

Docker comes with **built-in** (or native) drivers:

- **bridge** — Single-host local networks
- **host** — Container uses the host’s network stack directly (no isolation)
- **overlay** — Multi-host networks for Swarm (uses VXLAN)
- **macvlan** — Give containers real LAN IPs directly
- **ipvlan** — Similar to macvlan but more efficient; assigns IPs without creating new MACs
- **none** — Disable all networking for the container

> **Note:** You can also use third-party drivers for more advanced setups.

---

## 1. Bridge Network (Default & Most Common)

**What it is**
- A **private virtual LAN** for a single Docker host.
- Docker creates it automatically (**bridge**).
- Containers connected to a **bridge** can **communicate with each other**.
- **NAT** (Network Address Translation) is used when accessing the outside world.

### Example:
Let’s say you want to create a Docker network called **app-net** using the bridge driver.

```bash
docker network create --driver bridge app-net
```

**What happens internally?**
1. Docker asks **libnetwork** to create a network
2. **Libnetwork** → `“OK, I will manage this network in the control plane.”`
4. **Libnetwork** calls the selected driver

- It tells the **bridge driver**:
  * Create a bridge network named **app-net**

- The **bridge driver** performs the actual work (data plane):
  * Create a **Linux bridge** (like br-xxxxx)
  * Assign a **subnet** (e.g., 172.18.0.0/16)
  * Create **routing rules**
  * Set **NAT** so containers can reach the internet

**When you run a container on that network**
```
docker run -d --network app-net --name web nginx
```
The **bridge driver** will:
* Create a **veth pair**
* Attach one end to the container **sandbox**
* Connect the other end to the **bridge** (br-xxxxx)
* Assign an **IP** (e.g., 172.18.0.2) from that subnet
* Allow **container-to-container** communication on that bridge
