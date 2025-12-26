`docker volume inspect`
# docker volume inspect

**Usage**

```bash
docker volume inspect [OPTIONS] VOLUME [VOLUME...]
```

## Description

Displays **detailed low-level information** about one or more Docker volumes in **JSON format**.
This command is commonly used by **system administrators and DevOps engineers** to:

- Identify **where volume data is stored on the host**
- Verify **volume driver, scope, and labels**
- Troubleshoot **storage-related container issues**
- Extract specific metadata using formatted output

---

## Options

* `--format , -f` – Format the output using a **Go template** to extract specific fields

---

## 1. Basic Volume Inspection

### 1.1 Inspect a single Docker volume

```bash
docker volume inspect mydata
```

***Output:** Displays full **JSON metadata** of the volume.*\
***Purpose:** View **driver**, **mountpoint**, **scope**, and **creation details**.*


### 1.2 Inspect multiple Docker volumes at once

```bash
docker volume inspect vol1 vol2 vol2
```

***Output:** JSON array containing details of **all specified volumes**.*


---

## 2. Formatted Output

### 2.1 Display only the volume mountpoint

```bash
docker volume inspect --format '{{ .Mountpoint }}' mydata
```

***Output:** Host filesystem path only.*


### 2.2 Display only the volume driver

```bash
docker volume inspect -f '{{ .Driver }}' mydata
```

***Purpose:** Verify storage backend in **production environments**.*


### 2.3 Display volume name and mountpoint together

```bash
docker volume inspect -f 'Name={{ .Name }}  Mount={{ .Mountpoint }}' mydata
```

***Purpose:** Generate **human-readable output** for reports or audits.*

---

## 3. Label & Metadata Inspection

### 3.1 Inspect volume labels

```bash
docker volume inspect -f '{{ .Labels }}' mydata
```

***Purpose:** Validate **environment tags**, ownership, or application mapping.*



### 3.2 Inspect creation time of a volume

```bash
docker volume inspect -f '{{ .CreatedAt }}' mydata
```

***Purpose:** Track **volume lifecycle** and identify old or unused volumes.*

---

## 4. Swarm & Advanced Storage Validation

### 4.1 Check volume scope in Swarm environments

```bash
docker volume inspect -f '{{ .Scope }}' mydata
```

***Purpose:** Ensure correct **data accessibility across nodes**.*

**Reminder**
• `local` volumes are **node-specific**
• `global` volumes are **shared across the cluster**

---

## Summary

**`docker volume inspect` is most commonly used to:**

1. Verify **where data lives**
2. Troubleshoot **container storage issues**
2. Extract **specific metadata** for scripts
3. Validate **driver, scope, and labels**
5. Support **backup and audit workflows**


---