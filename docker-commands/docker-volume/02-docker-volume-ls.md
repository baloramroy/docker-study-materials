`docker volume ls`
# docker volume ls

**Usage**
```bash
docker volume ls [OPTIONS]
```
**Aliases**
```
docker volume list
```

## Description
**List all the volumes known to Docker**. You can **filter** using the `-f` or `--filter` flag. Refer to the filtering section for more information about available filter options.

---

## Used Options

* **`-a, --all`** – Show **all volumes**, including dangling (unused) ones
* **`-f, --filter`** – Filter volumes based on **criteria** (name, label, dangling)
* **`--format`** – Customize output using **Go templates** (automation-friendly)
* **`-q, --quiet`** – Display **only volume names** (scripting & cleanup)

---

## 1. Basic Volume Listing

### 1.1 List all Docker volumes

```bash
docker volume ls
```

_**Output:** Displays volume **DRIVER** and **VOLUME NAME**._\
_**Purpose:** Get a quick overview of all existing volumes on the host._


### 1.2 List all volumes

```bash
docker volume ls --all
```

_**Output:** Lists **used + unused** volumes._


---

## 2. Filtering Volumes

### 2.1 Filter volumes by name

```bash
docker volume ls --filter name=app
```

_**Output:** Shows volumes with **“app”** in their name._

---

### 2.2 Filter volumes by label

```bash
docker volume ls --filter label=env=prod
```

***Output:** Displays volumes tagged with **env=prod**.*

### 2.3 Filter dangling (unused) volumes

```bash
docker volume ls --filter dangling=true
```

***Output:** Shows volumes **not attached** to any container.*\
***Purpose:** Identify volumes safe for **cleanup** before pruning.*

---

## 3. Script-Friendly Output

### 3.1 Show only volume names

```bash
docker volume ls -q
```

***Output:** Prints **only volume names**, one per line.*\
***Purpose:** Ideal for **shell scripts**, loops, and automation.*

---

### 3.2 Combine quiet mode with filters

```bash
docker volume ls -q --filter dangling=true
```

***Output:** List of unused volume names only.*\
***Purpose:** Used before running **`docker volume rm`** safely.*

---

## 4. Custom Formatting (Monitoring & Reporting)

### 4.1 Custom table format

```bash
docker volume ls --format "table {{.Name}}\t{{.Driver}}"
```

***Output:** Clean, readable table with **Name** and **Driver**.*\
***Purpose:** Generate **human-friendly reports**.*

---

### 4.2 JSON-like output for tooling

```bash
docker volume ls --format '{{json .}}'
```

***Output:** One JSON object per volume.*\
***Purpose:** Integration with **monitoring tools** and parsers.*

---

