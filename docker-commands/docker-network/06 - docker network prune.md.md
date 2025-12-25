# `docker network prune`

## Description

The `docker network prune` command is used to **remove all unused Docker networks**.
Unused networks are those **not referenced by any running or stopped containers**.

This command is commonly used for **routine cleanup** to free system resources.


- Usage

  ```bash
  docker network prune [OPTIONS]
  ```

---

## 1. Prune All Unused Networks

### 1.1 Removes **all unused Docker networks**.

```bash
docker network prune
```

**Output:**\
*Prompts for confirmation: `Are you sure you want to continue? (y/N)`*\
*Removes all networks not used by any container*

**Purpose**\
*Routine cleanup*\
*Frees unused Docker network resources*

---

## 2. Force Prune Without Confirmation

### 2.1 Skips the interactive confirmation prompt.

```bash
docker network prune -f
```

**Output:**\
*Immediately removes all unused networks*\
*No confirmation prompt*

**Use Case:**\
*Automation scripts*\
*CI/CD pipelines*\
*Non-interactive environments*

---

## 3. Prune Networks by Label

### 3.1 Remove Networks With a Specific Label

```bash
docker network prune --filter "label=env=dev"
```

**Output**\
*Removes only networks labeled `env=dev`*

**Use Case**\
*Clean up development or testing networks*\
*Keep production networks intact*



### 3.2 Remove Networks Without a Specific Label

```bash
docker network prune --filter "label!=env=prod"
```

**Output**\
Removes all unused networks **except** those labeled `env=prod`

**Use Case**\
*Protect production networks*\
*Clean everything else safely*

---

## 4. Prune Networks Based on Creation Time

Docker allows pruning based on **age**, using the `until` filter.


### 4.1 Prune Networks Older Than 24 Hours

```bash
docker network prune --filter "until=24h"
```

**Output**\
*Removes unused networks created more than **24 hours** ago*


### 4.2 Prune Networks Older Than 72 Hours (3 Days)

```bash
docker network prune --filter "until=72h"
```

**Output**\
*Removes unused networks older than 3 days*

**Use Case**\
*Periodic cleanup*\
*Prevent accumulation of old networks*

---

## 5. Combine Multiple Filters

You can apply **multiple filters together** for precise cleanup.

**Example**\
Remove networks that:
- Are labeled `type=temp`
- Are older than 24 hours
- Are unused

### 5.1 Prune networks that are older than 24h and labeled "temp

```bash
docker network prune \
  --filter "label=type=temp" \
  --filter "until=24h"
```

**Output**\
*Removes only temporary, old, unused networks*

**Use Case**\
*Advanced cleanup strategies*
*Environment-specific automation*

---
