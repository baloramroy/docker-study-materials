# Docker Network Command Overview

## Description

Manage Docker networks. You can use **subcommands** to **create, inspect, list, remove, prune, connect,** and **disconnect** networks.

-   **Usage**
  
	```bash
	docker network [COMMAND]
	```

## Subcommands

-   **Connect a container to a network**
    ```bash
  	docker network connect
  	```
    
-   **Create a network**
	```bash
	docker network create
	```

-   **Disconnect a container from a network**
	```bash
	docker network disconnect
	```
	
-   **Display detailed information on one or more networks**
    ```bash
	docker network inspect
	```
	
-   **List networks**
    ```bash
	docker network ls
	```
    
-   **Remove all unused networks**
    ```bash
	docker network prune
	```
    
-   Remove one or more networks
	```bash
	docker network rm
	```
