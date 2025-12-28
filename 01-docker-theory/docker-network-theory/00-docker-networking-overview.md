# Docker Networking Overview

## Basic Concept

At a high level, **Docker networking** is built around three main components:

- **Container Network Model (CNM)**\
  This is the **blueprint** or design plan for how Docker networks should work.

- **Libnetwork**\
  This is the actual **code** that follows the **CNM** design.\
  It’s **open-source** and used by Docker to manage networking.

- **Drivers**\
  These are tools that create different types of networks, like **overlay, bridge networks**.
  
---

## Explanation of those Three components

### 1. The Container Network Model (CNM)

Docker networking is designed using something called the **Container Network Model (CNM)**. It’s like the **blueprint** for how networking works in containers.

It’s built on **three** key parts:

**i. Sandbox**

- This is the **network environment** inside each container.
- It includes things like **IP addresses**, **routing tables**, **ports**, and **DNS**.
- Every container gets its own **sandbox**.

**ii. Endpoint**

- Think of it like a **virtual network cable**.
- It **connects the sandbox to a network**.
- It works like a normal **network interface** (like a network card).

**iii. Network**

- This is a **virtual switch (software-based)**.
- It allows containers to **talk to each other** if they’re connected to the same **network**.
- It can also **isolate containers** on different networks.

**Real-life Example**

* **Container A** has one **endpoint**, connected to **Network A**.
* **Container B** has two **endpoints**: one on **Network A**, one on **Network B**.
* Because both **Container A** and **Container B** are on **Network A**, they can **talk to each other**.
* But the two **interfaces** inside **Container B** can’t talk to each other because they’re on **different networks**.
* Even if containers are on the same **Docker host**, they’re still **isolated**. They can only communicate through a **Docker-defined network**.

---

### 2. Libnetwork

**Libnetwork** is the system that handles all networking in Docker.
Before it existed, networking code was inside the **Docker daemon**, which made Docker complicated.
So Docker moved all networking logic into a separate tool called **libnetwork**.

**What Libnetwork Does**

* Implements the entire **CNM** architecture.
* Manages the **control plane** (**network creation**, **configuration**, **management**).
* **Service discovery**: Lets containers find each other by name.
* **Load balancing**: Handles load balancing between containers.
* **Management APIs**: Provides APIs so Docker and other tools can control networking.

---

### 3. Drivers

**Drivers** are the tools that actually build and run the actual network creation and connecting job.

Docker networking consists of **two** layers:

**i. Control Plane – Managed by libnetwork**\
Handles mostly supervised **network creation**, **configuration**, and **management**.

**ii. Data Plane – Managed by drivers**\
Responsible for actually **creating networks**, **connecting containers**, and **isolating them**.

---

