
# `docker network ls`

### Description

Lists all the networks the Engine **daemon** knows about. This includes the networks that span across **multiple hosts** in a cluster.

- Usage
  ```
  docker network ls [OPTIONS]
  ```

- Aliases
  ```
  docker network list
  ```

#

### List All Networks

- List all available Docker networks

  ```
  docker network ls
  ```

  > **Output:** Shows all networks including bridge, host, none, and user-defined networks.

#

### Filter by Network Name

- List networks matching a name

  ```
  docker network ls --filter "name=prod"
  ```
  
  > **Output:** Shows networks whose names contain `prod`.

#

### Filter by Driver

- List only **bridge** networks

  ```
  docker network ls --filter "driver=bridge"
  ```
  
  > **Output:** Displays networks using the bridge driver.

- List only overlay networks

  ```
  docker network ls --filter "driver=overlay"
  ```
  
  > **Output:** Displays overlay networks (common in Swarm/Kubernetes).

#

### Filter by Scope

- List only local-scope networks

  ```
  docker network ls --filter "scope=local"
  ```
  
  > **Output:** Shows networks limited to a single node.
  
  > **purpose:** Identify networks not shared across clusters.

- List only swarm-scope networks

  ```
  docker network ls --filter "scope=swarm"
  ```
  
  > **Output:** Shows multi-node Swarm networks.

#

### Filter by Network Type (Built-in vs User-Defined)

- List only built-in networks

  ```
  docker network ls --filter "type=builtin"
  ```
  
  > **Output:** Shows bridge, host, and none.
  
  > **purpose:** Quickly identify default networks.

- List only user-created networks

  ```
  docker network ls --filter "type=custom"
  ```
  
  > **Output:** Displays networks created by admins or apps.

#

### Display Only Network IDs

- List only network IDs

  ```
  docker network ls -q
  ```
  
  > **Output:** Shows only IDs of all networks.
  
  > **purpose:** Useful for bulk removal or scripting.

- Example: remove all unused custom networks

  ```
  docker network rm $(docker network ls -q)
  ```

  > **Output:** Removes all networks (except those in use).

#

### Combining Multiple Filters

- Filter networks by name + driver

  ```
  docker network ls --filter "name=web" --filter "driver=bridge"
  ```
  
  > **Output:** Shows only bridge networks whose name contains `web`.

#
