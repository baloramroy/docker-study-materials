`docker run`

# Docker Run

## Description

Creates and starts a **new container** from a specified Docker image.

`docker run` is a **core Docker command** that combines multiple actions into one step:

1. Pulls the image (if not already available locally)
2. Creates a new container
3. Applies configuration (network, volumes, environment variables, etc.)
4. Starts the container

This command is **widely used in development and production** to run applications, services, databases, and microservices.

- **Usage**  
    `docker container run [OPTIONS] IMAGE [COMMAND] [ARG...]`  
- **Aliases**  
    `docker run`

---

## Options

**Container Lifecycle & Mode**

* `-d, --detach` – Run container in **background (detached mode)**
* `--rm` – Automatically **remove container** when it stops
* `--name` – Assign a **custom container name**
* `--restart` – Define **restart policy** (`no`, `on-failure`, `always`, `unless-stopped`)

#

**Networking**

* `-p, --publish` – **Map ports** (host:container)
* `--network` – Connect container to a **specific Docker network**
* `--hostname` – Set container **hostname**

#

**Storage & Volumes**

* `-v, --volume` – Attach **volume or bind mount**
* `--mount` – Advanced, **explicit mount syntax**
* `--tmpfs` – Mount **temporary in-memory filesystem**

#

**Environment & Configuration**

* `-e, --env` – Set **environment variables**
* `--env-file` – Load environment variables from a **file**
* `-w, --workdir` – Set **working directory** inside container

#

**Resources & Limits**

* `--memory` – Set **memory limit**
* `--cpus` – Limit **CPU usage**
* `--pids-limit` – Limit number of **process IDs**

#

**User & Security**

* `-u, --user` – Run container as a **specific user**
* `--privileged` – Give container **extended privileges**
* `--cap-add / --cap-drop` – Add or remove **Linux capabilities**

#

**Execution & Control**

* `-i, --interactive` – Keep **STDIN open**
* `-t, --tty` – Allocate a **pseudo-TTY**
* `--entrypoint` – Override the image **entrypoint**
* `--command` – Override default **CMD** of the image


---

## Basic Container Operations

- **Run container in background**  
    `docker run -d nginx`  
    **Output:** Starts container in detached mode and prints container ID.  
    **Purpose:** Run container while keeping the terminal free.

- **Run container with a custom name**  
    `docker run --name mynginx nginx`  
    **Output:** Starts container with the name mynginx.  
    **Purpose:** Easier identification and management.

---

## Networking Options

- **Map container port to host port**  
    `docker run -p 8080:80 nginx`  
    **Output:** Application becomes available on host port 8080.  
    **Purpose:** Expose container service externally.

- **Expose multiple ports**  
    `docker run -p 8080:80 -p 8443:443 nginx`  
    **Output:** Both ports become accessible.  
    **Purpose:** Useful for HTTP & HTTPS services.

- **Connect container to a custom network**  
    `docker run --network=my_net nginx`  
    **Output:** Container joins my_net network.  
    **Purpose:** Communication between microservices.

---

## Volume & Storage Options
- **Bind mount local directory**  
  `docker run -v /host/data:/container/data nginx`  
  **Output:** Mounts host folder into the container.  
  **Purpose:** Persistent storage, file sharing.

- **Mount named volume**  
  `docker run -v myvol:/var/lib/mysql mysql:8`  
  **Output:** Database stores data in myvol volume.  
  **Purpose:** Persistent DB storage.

---

## Environment Variable Options

- **Set one environment variable**  
    `docker run -e ENV=prod nginx`  
    **Output:** ENV variable available inside container.  
    **Purpose:** App configuration.

- **Load env variables from file**  
    `docker run --env-file env.list nginx`  
    **Output:** Loads all variables from env.list.  
    **Purpose:** Clean, organized production configuration.

---

## Resource & Limit Controls

- **Limit CPU**  
    `docker run --cpus="1.5" nginx`  
    **Output:** Limits CPU usage to 1.5 cores.  
    **Purpose:** Resource control.

- **Limit memory**  
    `docker run -m 512m nginx`  
    **Output:** Allocates 512 MB RAM limit.  
    **Purpose:** Prevent memory overuse.

- **Memory + swap**  
    `docker run -m 1g --memory-swap=2g nginx`  
    **Output:** Memory capped at 1GB + swap limit 2GB.  
    **Purpose:** Strict resource boundaries.

---

## Restart Policies

- **Always restart**  
    `docker run --restart=always nginx`  
    **Output:** Restarts automatically whenever it stops.  
    **Purpose:** Critical production containers.

- **Restart unless manually stopped**  
    `docker run --restart=unless-stopped nginx`  
    **Output:** Restarts automatically except after manual stop.  
    **Purpose:** Long-running services.

---

## User & Security Options

- **Run as specific user**  
    `docker run -u 1000 nginx`  
    **Output:** Runs processes as UID 1000.  
    **Purpose:** Avoid running as root.

- **Read-only filesystem**  
    `docker run --read-only nginx`  
    **Output:** Container filesystem becomes read-only.  
    **Purpose:** Security hardening.

---

## Logging & Debugging
- **Run container interactively**  
    `docker run -it centos bash`  
    **Output:** Opens bash shell inside container.  
    **Purpose:** Debugging and troubleshooting.

- **One-time command with auto-remove**  
    `docker run --rm alpine echo "hello"`  
    **Output:** Prints hello, container removed.  
    **Purpose:** Lightweight tests or tasks.

---

## Entry & Execution Options

- **Override entrypoint**  
    `docker run --entrypoint /bin/bash ubuntu`  
    **Output:** Starts bash instead of default entrypoint.  
    **Purpose:** Debug image or override behavior.

- **Pass command arguments**  
    `docker run ubuntu echo "Hello World"`  
    **Output:** Prints Hello World.  
    **Purpose:** Run simple commands in containers.

---