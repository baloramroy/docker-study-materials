
# `docker network ls`

### Description

Lists all the networks the Engine **daemon** knows about. This includes the networks that span across **multiple hosts** in a cluster.

- **Usage**
  ```
  docker network ls [OPTIONS]
  ```

- **Aliases**
  ```
  docker network list
  ```

---

### 1. List All Networks

**1.1 List all available Docker networks**

```bash
docker network ls
```

_**Output:** Shows all networks including bridge, host, none, and user-defined networks._

---

### 2. Filter by Network Name

**2.1 List networks matching a name**

```bash
docker network ls --filter "name=prod"
```

_**Output:** Shows networks whose names contain `prod`._

---

### 3. Filter by Driver

**3.1 List only **bridge** networks**

```bash
docker network ls --filter "driver=bridge"
```

_**Output:** Displays networks using the bridge driver._


**3.2 List only overlay networks**

```bash
docker network ls --filter "driver=overlay"
```

_**Output:** Displays overlay networks (common in Swarm/Kubernetes)._

#

### 4. Filter by Scope

**4.1 List only local-scope networks**

```bash
docker network ls --filter "scope=local"
```

_**Output:** Shows networks limited to a single node._\
_**purpose:** Identify networks not shared across clusters._


**4.2 List only swarm-scope networks**

```bash
docker network ls --filter "scope=swarm"
```

_**Output:** Shows multi-node Swarm networks._

---

### 5. Filter by Network Type (Built-in vs User-Defined)

**5.1 List only built-in networks**

```bash
docker network ls --filter "type=builtin"
```

_**Output:** Shows bridge, host, and none._\
_**purpose:** Quickly identify default networks._


**5.2 List only user-created networks**

```bash
docker network ls --filter "type=custom"
```

_**Output:** Displays networks created by admins or apps._

---

### 6. Display Only Network IDs

**6.1 List only network IDs**
```bash
docker network ls -q
```

_**Output:** Shows only IDs of all networks._\
_**purpose:** Useful for bulk removal or scripting._


**6.2 Example: Remove all unused custom networks**
```
docker network rm $(docker network ls -q)
```

_**Output:** Removes all networks (except those in use)._

---

### 7. Combining Multiple Filters

**7.1 Filter networks by name + driver**

```bash
docker network ls --filter "name=web" --filter "driver=bridge"
```

_**Output:** Shows only bridge networks whose name contains `web`._

---
