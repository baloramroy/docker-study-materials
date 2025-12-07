# Docker Compose Overview

## What is Docker Compose?
Docker Compose is a tool that lets you define and run multiple containers (multi-container applications) using a single YAML file called **docker-compose.yml**.
Instead of running containers one by one with long commands, you write everything in the **YAML** file and then start all services using:

```bash
docker compose up -d
```

Think of Docker Compose as:
* A **manager** for multiple Docker containers
* A way to **save your container setup as code**
* A tool to run services together easily (e.g., web + database + monitoring)

## How Docker Compose Works
Docker Compose works through **three** main steps:

### Step 1 – Define services in a YAML file
You create a **docker-compose.yml** file.
Inside it, you define **Top level element**:
* **Services** (containers)
* **Networks**
* **Volumes**
* **Environment variables**
* **Ports**
* **Dependencies** between services

**Example:**
```yaml
services:
  web:
    image: nginx
    ports:
      - "80:80"
  db:
    image: mysql
```
This file tells Docker:
>“Create one nginx container and one MySQL container.”

### Step 2 – Compose reads the file
When you run:

```bash
docker compose up
```
Docker Compose reads your YAML file and understands:
* Which images to **pull**
* What networks to **create**
* What volumes to **create**
* How many containers to **run**
* Which container **depends** on which

>Compose then creates everything **automatically**.


### Step 3 – Compose manages the lifecycle
Compose allows you to:
* **Start** containers
* **Stop** containers
* **Recreate** containers
* Check **logs**
* View **running** services

**Common commands:**
```bash
docker compose up -d
docker compose down
docker compose ps
docker compose logs -f
```

## Why Use Docker Compose?

### Reason 1 — Easy Multi-Container Setup
If your application needs:
* Backend
* Frontend
* MySQL
* Redis
* NGINX
* Prometheus
* Grafana
  
it would be **painful** to run all using individual `docker run` commands.

Compose allows you to start everything with **one command**:
```bash
docker compose up -d
```

### Reason 2 — Everything in One Configuration File
Your entire environment is stored in a **single file**: `docker-compose.yml`

**Benefits:**
* Easy to **share**
* Easy to **version-control**
* Easy to **understand** what’s running

### Reason 3 — Handles Networking Automatically
Compose automatically creates a **network** for your services so they can talk using service names:

**Example:**
`web → talks to → db`  
Using name → **db:3306**

>No need to manually create networks.

### Reason 4 — Handles Volumes Automatically
You can define **persistent storage** in the YAML file:
```yaml
volumes:
  db-data:
```
And attach it:
```yaml
services:
  db:
    volumes:
      - db-data:/var/lib/mysql
```
>Compose **creates and manages** this volume for you.

### Reason 5 — Easy Updates and Rebuilds
Updated your app code?
Then use:
```bash
docker compose up -d --build
```
>Compose **rebuilds and restarts** only what changed.

### Reason 6 — Consistency Across Environments
Same **docker-compose.yml** works on:
* Your laptop
* Another person’s laptop
* Your staging server
* Your production server
  
>This ensures your environment is **consistent**.

### Reason 7 — Faster Development
Compose supports:
* **Live code mounting** (bind mounts)
* **Environment files** (.env)
* **Command overrides**

>Great for everyday **development** work.
