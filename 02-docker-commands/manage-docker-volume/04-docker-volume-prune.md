`docker volume prune`
# docker volume prune

## Description

Removes **all unused Docker volumes** from the local system.
A volume is considered **unused** if it is **not referenced by any container** (running or stopped).

This command is commonly used for **disk cleanup**, **environment maintenance**, and **CI/CD housekeeping**.

**Warning:**\
Once removed, **volume data cannot be recovered**.

**Usage**

```bash
docker volume prune [OPTIONS]
```

---

## Options

* `-f, --force` – Skip the **confirmation prompt** and remove volumes **immediately**
* `--filter <key=value>` – Apply conditions to control which volumes are pruned

---

## 1. Basic Volume Cleanup

### 1.1 Prune unused volumes with confirmation

```bash
docker volume prune
```

***Output:** Prompts for confirmation and then lists deleted volumes.*\
***Purpose:** Safely clean up **unused volumes** after verifying the action.*


### 1.2 Prune unused volumes without confirmation

```bash
docker volume prune -f
```

***Output:** Immediately removes unused volumes without prompting.*\
***Purpose:** Used in **scripts**, **cron jobs**, and **CI/CD pipelines**.*

---

## 2. Filter-Based Volume Cleanup

### 2.1 Prune volumes created before a specific time

```bash
docker volume prune --filter "until=24h"
```

***Output:** Removes volumes unused for more than **24 hours**.*\
***Purpose:** Clean up **old unused volumes** while keeping recent ones.*


### 2.2 Prune volumes unused for more than 7 days

```bash
docker volume prune --filter "until=168h"
```

***Output:** Deletes volumes unused for over **7 days**.*

---

## Important Operational Notes

• **Running containers are never affected**
• **Stopped containers still protect their volumes**
• Only **dangling (unused) volumes** are removed
• Works **only on local Docker volumes**, not external storage
• Always inspect volumes before pruning in **production systems**

---

## Best Practice Recommendation

- Use `docker volume ls` and `docker volume inspect` **before pruning**
- Prefer `--filter "until=..."` in **production environments**
- Use `-f` only in **automation or controlled servers**

---


