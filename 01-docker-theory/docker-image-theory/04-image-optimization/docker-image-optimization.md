# Phase 4 — Docker Image Optimization

## Part 1: Understanding Image Optimization

### 1. Objective

In Phase 3, you learned how `.dockerignore` prevents unnecessary files from entering the build context.

Now we move one level deeper:

> **How do we build a Docker image that contains only what the application actually needs?**

Image optimization is not simply about making the image smaller. A well-designed image should be:

* Small enough to reduce transfer and storage costs.
* Fast to build and pull.
* Free of unnecessary software and files.
* Suitable for production.
* Easier to scan and maintain.
* Designed to run with the minimum required privileges.

For CI/CD, this matters because your pipeline may build and push the image repeatedly.

```text
Code change
    ↓
CI pipeline
    ↓
Docker build
    ↓
Image
    ↓
Registry
    ↓
Deployment
```

If the image is unnecessarily large, that cost is repeated throughout the pipeline.

---

## 2. The Three Main Areas of Optimization

We'll focus on three areas:

```text
Image Optimization
       │
       ├── 1. Choose the right base image
       │
       ├── 2. Reduce unnecessary image contents
       │
       └── 3. Run as a non-root user
```

There are other optimization techniques, particularly around caching and multi-stage builds, but we'll handle those separately because they deserve their own concepts.

---

## 3. Base Image Selection

Every Docker image normally starts with a base image:

```dockerfile
FROM python:3.12
```

The base image contributes directly to the final image.

For example, Python provides several commonly used variants:

```text
python:3.12
python:3.12-slim
python:3.12-alpine
```

These are not simply different tags for exactly the same image.

They represent different underlying environments and trade-offs.

### Standard image

```dockerfile
FROM python:3.12
```

Provides a relatively complete Debian-based environment.

Advantage:

* More system packages and tools are available.

Disadvantage:

* Larger image.
* More software than many applications actually need.

### Slim image

```dockerfile
FROM python:3.12-slim
```

Provides a more minimal Debian-based environment.

This is often a good starting point for production Python applications because you get a smaller environment without changing the underlying Linux family completely.

### Alpine

```dockerfile
FROM python:3.12-alpine
```

Uses Alpine Linux.

It can be significantly smaller, but **smaller does not automatically mean better**.

Some Python packages depend on native libraries or compilation environments, and Alpine's use of musl libc can introduce compatibility or build considerations.

So don't choose a base image simply because its advertised size is smaller.

The correct question is:

> **What is the smallest appropriate base image for this application?**

---

## 4. A Simple Comparison

Consider:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

We could first test whether:

```dockerfile
FROM python:3.12-slim
```

works for the application.

If it does, we may reduce the image significantly without changing the application's behavior.

The workflow should be:

```text
Choose base image
       ↓
Build
       ↓
Test
       ↓
Scan
       ↓
Compare
```

Not:

```text
Smallest image available
       ↓
Use it immediately
```

---

## 5. Image Contents Matter

Consider this Dockerfile:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY . .

RUN pip install -r requirements.txt

CMD ["python", "app.py"]
```

Suppose the build context contains:

```text
app.py
requirements.txt
README.md
tests/
docs/
sample-data/
debug.log
```

Even though you used a relatively small base image, you may still end up putting unnecessary files into the image.

That's why `.dockerignore` and image optimization work together:

```text
.dockerignore
      ↓
Smaller build context
      ↓
Cleaner COPY
      ↓
Smaller final image
```

For example:

```dockerignore
.git/
*.log
__pycache__/
.pytest_cache/
README.md
docs/
```

This is one reason Phase 3 came before Phase 4.

---

## 6. Don't Install Unnecessary Packages

Another common problem is installing tools into the final image that the application doesn't need.

For example:

```dockerfile
RUN apt-get update && \
    apt-get install -y \
        curl \
        vim \
        git \
        wget \
        net-tools
```

If the application doesn't require these tools at runtime, they add unnecessary software to the image.

A production image should generally contain:

```text
Application
+
Runtime dependencies
+
Required system libraries
```

rather than:

```text
Application
+
Runtime dependencies
+
Developer tools
+
Debugging tools
+
Build tools
+
Unnecessary packages
```

This becomes particularly important for security scanning.

Every additional package is additional software that potentially needs to be maintained and scanned.

---

## 7. Package Manager Cleanup

If you install Debian packages, package metadata can remain behind.

For example:

```dockerfile
RUN apt-get update && \
    apt-get install -y curl
```

A common production pattern is:

```dockerfile
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*
```

There are two useful ideas here:

### `--no-install-recommends`

It avoids installing packages that are merely recommended rather than required.

### `/var/lib/apt/lists/*`

APT stores package metadata there.

After installation, that metadata generally isn't needed in the runtime image, so it can be removed.

This is a small example of an important principle:

> **Don't leave temporary build/package data in the final image.**

---

## 8. Combine Related Commands

Consider:

```dockerfile
RUN apt-get update
RUN apt-get install -y curl
RUN rm -rf /var/lib/apt/lists/*
```

This creates multiple Dockerfile build steps.

A better pattern is:

```dockerfile
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*
```

Why?

Because the installation and cleanup happen within the same build step.

This avoids leaving intermediate filesystem changes behind in separate layers.

You don't need to aggressively combine every `RUN` instruction, though. Readability and caching also matter.

---

## 9. Non-Root Containers

This is an important part of image optimization from a **security** perspective.

By default, many containers run as `root`.

For example:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY . .

CMD ["python", "app.py"]
```

The application may run as root inside the container.

That's unnecessary for many applications.

A better design is to create a dedicated user:

```dockerfile
FROM python:3.12-slim

RUN useradd --create-home appuser

WORKDIR /app

COPY . .

USER appuser

CMD ["python", "app.py"]
```

Now the application runs as:

```text
appuser
```

instead of:

```text
root
```

---

## 10. Why Non-Root Matters

Containers provide isolation, but you should still follow the principle of least privilege.

If the application process runs as root and the application is compromised, the attacker initially has root privileges **inside the container**.

Running as a non-root user reduces those privileges.

Conceptually:

```text
Bad:

Container
└── root
    └── application


Better:

Container
└── appuser
    └── application
```

This doesn't make the container automatically secure, but it removes one unnecessary privilege.

---

## 11. File Ownership

There's one detail to watch when using `USER`.

Suppose:

```dockerfile
RUN useradd --create-home appuser

WORKDIR /app

COPY . .

USER appuser
```

The files copied by `COPY` may be owned by `root`, depending on the build and Dockerfile configuration.

If the application needs to write to those files, this can cause permission problems.

For example:

```text
/app
├── app.py
└── data/
```

If `appuser` needs to write to `data/`, ensure the directory is appropriately owned or writable.

One common pattern is:

```dockerfile
RUN useradd --create-home appuser

WORKDIR /app

COPY --chown=appuser:appuser . .

USER appuser
```

Now the copied files are assigned to the application user.

But don't blindly make everything writable.

Use the minimum permissions the application actually needs.

---

## 12. Putting the Ideas Together

Let's create a cleaner Dockerfile.

Instead of:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY . .

RUN apt-get update && apt-get install -y curl vim git

RUN pip install -r requirements.txt

CMD ["python", "app.py"]
```

we can aim for:

```dockerfile
FROM python:3.12-slim

RUN useradd --create-home appuser

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY --chown=appuser:appuser . .

USER appuser

CMD ["python", "app.py"]
```

There are several improvements here:

| Change                               | Reason                                        |
| ------------------------------------ | --------------------------------------------- |
| `python:3.12-slim`                   | Smaller appropriate base image                |
| `requirements.txt` copied separately | Helps build caching                           |
| `--no-cache-dir`                     | Avoids retaining pip's download cache         |
| No unnecessary tools                 | Keeps runtime image focused                   |
| `appuser`                            | Avoids running application as root            |
| `COPY --chown`                       | Gives application files appropriate ownership |

Notice that one line:

```dockerfile
COPY requirements.txt .
```

also relates to **build cache optimization**.

We already introduced caching in Phase 1, but we'll revisit its deeper optimization techniques later.

---

## 13. Important: Smaller ≠ Automatically Better

This is worth remembering.

Suppose you have:

```text
Image A: 50 MB
Image B: 100 MB
```

It doesn't automatically mean Image A is the better production image.

You should consider:

```text
Size
Compatibility
Security
Maintainability
Build performance
Application requirements
```

For example, forcing an application onto an extremely minimal base image may create complicated dependency problems.

The goal is:

> **Minimal appropriate image, not minimum possible image.**

---

## 14. Practical Exercise

Now let's test this properly.

We'll use the application from Phase 3.

Your project:

```text
docker-image-lab/
├── app.py
├── requirements.txt
├── Dockerfile
└── .dockerignore
```

### Step 1 — Build the current image

Use your existing Dockerfile and build:

```bash
docker build --no-cache -t image-opt-lab:original .
```

Then check:

```bash
docker image ls image-opt-lab
```

Record the image size.

---

### Step 2 — Change the base image

Change:

```dockerfile
FROM python:3.12
```

to:

```dockerfile
FROM python:3.12-slim
```

Build:

```bash
docker build --no-cache -t image-opt-lab:slim .
```

Compare:

```bash
docker image ls image-opt-lab
```

---

### Step 3 — Test the optimized image

Don't assume the smaller image works.

Run:

```bash
docker run --rm image-opt-lab:slim
```

Verify that the application still works.

This is an important CI/CD principle:

```text
Optimization
     ↓
Build
     ↓
Test
     ↓
Accept optimization
```

not:

```text
Smaller image
     ↓
Assume it's better
```

---

### Step 4 — Add a non-root user

Modify your Dockerfile:

```dockerfile
RUN useradd --create-home appuser
```

and:

```dockerfile
USER appuser
```

Build:

```bash
docker build --no-cache -t image-opt-lab:nonroot .
```

Then verify the user:

```bash
docker run --rm image-opt-lab:nonroot whoami
```

Expected:

```text
appuser
```

You can also verify with:

```bash
docker run --rm image-opt-lab:nonroot id
```

---

## 15. Phase 4 Checkpoint

Before moving to Phase 5, you should be able to explain these without looking back:

1. Why might `python:3.12-slim` be preferable to `python:3.12`?
2. Why isn't the smallest possible base image automatically the best choice?
3. Why should unnecessary packages such as `vim` and `git` generally not be installed in a runtime image?
4. Why do we remove `/var/lib/apt/lists/*` after installing Debian packages?
5. Why is `pip install --no-cache-dir` useful?
6. Why should production containers preferably run as a non-root user?
7. What problem can occur when files are owned by `root` but the application runs as `appuser`?
8. How do `.dockerignore` and image optimization complement each other?
9. Why should you test an optimized image instead of judging it only by its size?

### One thing to notice

We deliberately **did not go deep into multi-stage builds here**.

Multi-stage builds are one of the most important image-optimization techniques, but they introduce a different mental model:

```text
Build environment
       ↓
Build application
       ↓
Production environment
       ↓
Copy only required artifacts
```

That's the subject of **Phase 5**, and we'll treat it properly rather than squeezing it into this lesson.
