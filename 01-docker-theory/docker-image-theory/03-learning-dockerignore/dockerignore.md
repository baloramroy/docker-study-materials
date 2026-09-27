# Phase 3 — `.dockerignore`

## Learning Docker Ignore

### Where We Are

**Phase 1 callback:** we already know the build context is the set of files Docker receives when you run:

```bash
docker build -t my-app:1.0 .
```

And we know that being *in* the build context doesn't automatically mean a file ends up *in* the image. That's what Dockerfile instructions such as `COPY` decide.

But there's a problem we haven't solved yet:

**What if you don't even want certain files available to the Docker build in the first place?**

That's what `.dockerignore` is for.

---

## 1. The Problem `.dockerignore` Solves

Imagine a real project directory:

```text
my-app/
├── app.py
├── requirements.txt
├── Dockerfile
├── .git/
├── node_modules/
├── venv/
├── .env
└── debug.log
```

When you use:

```bash
docker build -t my-app:1.0 .
```

the directory represented by `.` is the build context.

Files inside that context are available to the build unless they are excluded by `.dockerignore`.

That means files such as:

```text
.git/
node_modules/
venv/
.env
debug.log
```

may be available to the builder even though the application doesn't need them.

This is unnecessary and, in some cases, risky.

---

## 2. Why This Actually Matters

### It can increase build overhead

A project can contain large directories that have nothing to do with the image.

For example:

```text
node_modules/
.git/
venv/
```

A large build context can increase the amount of data the builder needs to process and can slow down builds, particularly in CI/CD environments.

```text
Large project
     ↓
Large build context
     ↓
More build overhead
     ↓
Potentially slower CI pipeline
```

The exact behavior depends on the Docker builder, but keeping the build context small is a good practice.

### It can expose sensitive files

Consider:

```text
.env
```

which might contain:

```text
DATABASE_PASSWORD=...
API_KEY=...
SECRET_KEY=...
```

You don't want sensitive files unnecessarily available to the Docker build.

`.dockerignore` helps prevent accidental inclusion of such files in the context.

**Important:** `.dockerignore` is not a replacement for proper secret management. Secrets should not be placed in Docker images or unnecessarily included in build contexts.

### It can prevent unwanted files from entering the image

Consider this Dockerfile:

```dockerfile
COPY . .
```

This tells Docker to copy the available build-context contents into the image.

If your context contains:

```text
.git/
.env
debug.log
venv/
```

and you haven't excluded them, `COPY . .` can copy them into the image.

That can unnecessarily increase the image size and put files into the image that don't belong there.

---

## 3. What `.dockerignore` Actually Does

`.dockerignore` is a plain-text file containing patterns that exclude files from the build context.

For example:

```text
.git
node_modules/
venv/
.env
*.log
```

The basic flow is:

```text
Project directory
       │
       │
       ▼
.dockerignore
       │
       │ Apply ignore rules
       ▼
Filtered build context
       │
       │ docker build
       ▼
Docker builder
       │
       ▼
Dockerfile instructions
```

The important distinction is:

```text
.dockerignore
→ controls what is available in the build context

COPY
→ controls what gets copied from that available context into the image
```

So if `.env` is excluded by `.dockerignore`:

```dockerfile
COPY . .
```

cannot copy `.env` because `.env` isn't available in the context presented to the build.

`.dockerignore` does **not** delete `.env` from your computer.

The file remains on your host exactly where it was.

---

## 4. Where Should `.dockerignore` Go?

For a standard Docker build, `.dockerignore` is placed at the root of the build context.

For example:

```text
my-app/
├── .dockerignore
├── Dockerfile
├── app.py
└── requirements.txt
```

When you run:

```bash
docker build -t my-app:1.0 .
```

the `.` means:

```text
my-app/
```

is the build context.

Therefore Docker uses:

```text
my-app/.dockerignore
```

The important point is:

**`.dockerignore` is associated with the build context, not simply with whichever directory happens to contain your Dockerfile.**

There are also Dockerfile-specific ignore-file mechanisms, but we don't need them for this phase.

---

## 5. Syntax Basics

`.dockerignore` patterns are similar in spirit to `.gitignore`.

You don't need to memorize every pattern rule yet. Focus on the common patterns you'll use in real projects.

### Ignore a specific file

```text
.env
```

This excludes `.env`.

You can also specify a path:

```text
config/production.env
```

### Ignore a directory

```text
.git/
venv/
node_modules/
```

These exclude those directories and their contents.

### Ignore files using `*`

```text
*.log
```

This excludes files ending in `.log`.

For example:

```text
debug.log
application.log
error.log
```

### Ignore nested matching files with `**`

Docker supports the `**` wildcard for matching directories at any depth.

For example:

```text
**/*.log
```

can match:

```text
debug.log
logs/app.log
services/api/debug.log
```

This is useful when you want a pattern to apply throughout a directory tree.

### Re-include with `!`

You can use `!` to re-include something that was previously excluded.

For example:

```text
*.log
!important.log
```

This means:

```text
all .log files → excluded
important.log → re-included
```

The order of matching rules matters.

For normal Docker projects, you will mostly use simple exclusions such as:

```text
.git/
.env
*.log
node_modules/
venv/
```

---

## 6. A Concrete Before/After

Suppose your Dockerfile is:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

Your project contains:

```text
my-app/
├── app.py
├── requirements.txt
├── Dockerfile
├── .git/
├── node_modules/
├── venv/
├── .env
└── debug.log
```

### Without `.dockerignore`

`COPY . .` can copy the available context contents into `/app`.

That could result in unwanted files such as:

```text
/app/.git/
/app/node_modules/
/app/venv/
/app/.env
/app/debug.log
```

### With `.dockerignore`

Create:

```text
.git/
node_modules/
venv/
.env
*.log
```

Now those files are excluded from the build context.

Therefore:

```dockerfile
COPY . .
```

cannot copy them.

Conceptually:

```text
Project
   │
   ├── app.py              → included
   ├── requirements.txt    → included
   ├── Dockerfile          → available to build
   ├── .git/               → excluded
   ├── node_modules/       → excluded
   ├── venv/               → excluded
   ├── .env                → excluded
   └── debug.log           → excluded
```

This is why `.dockerignore` and `COPY` work together.

---

## 7. Connection to CI/CD

This is particularly important for your CI/CD learning.

A typical pipeline might look like:

```text
Git repository
      │
      ▼
CI runner checks out project
      │
      ▼
Full working directory
      │
      │ docker build
      ▼
.dockerignore filters context
      │
      ▼
Docker builder
      │
      ▼
Docker image
```

A CI runner may have files that are useful to development but completely unnecessary for the Docker build:

```text
.git/
node_modules/
tests/
coverage/
*.log
local configuration
```

A properly designed `.dockerignore` keeps those files out of the build context.

This matters because CI/CD pipelines may build images repeatedly:

```text
Developer commit
      ↓
Pipeline
      ↓
Docker build
      ↓
Developer commit
      ↓
Pipeline
      ↓
Docker build
      ↓
... repeated many times
```

Keeping the context clean helps avoid unnecessary build overhead and reduces the chance of accidentally including sensitive or irrelevant files.

---

## 8. Practical Python `.dockerignore`

For a Python application, a reasonable starting point might be:

```text
# Git
.git/
.gitignore

# Environment files
.env
.env.*

# Python virtual environments
venv/
.venv/

# Python cache
__pycache__/
*.py[cod]

# Test and coverage cache
.pytest_cache/
.coverage

# Logs
*.log
logs/

# Local development files
.vscode/
.idea/
```

This is **not a universal template**.

You should decide what belongs in your particular build context.

For example, if your Docker build needs the `tests/` directory to execute tests inside the build, don't blindly exclude it.

The rule is simple:

> **Exclude files that the Docker build does not need.**

---

## 9. Hands-On Exercise

Now let's see `.dockerignore` working for ourselves.

Create:

```text
dockerignore-demo/
├── Dockerfile
├── app.py
├── secret.txt
└── notes.log
```

### `app.py`

```python
print("Hello from Docker!")
```

### `secret.txt`

```text
this should never end up in the image
```

### `notes.log`

```text
some log content
```

### Dockerfile

```dockerfile
FROM python:3.12

WORKDIR /app

COPY . .

CMD ["python", "app.py"]
```

---

### Step 1 — Build without `.dockerignore`

Make sure there is no `.dockerignore` file.

Build:

```bash
docker build -t ignore-demo:1.0 .
```

Then inspect `/app`:

```bash
docker run --rm --entrypoint ls ignore-demo:1.0 -la /app
```

You should see:

```text
app.py
secret.txt
notes.log
```

Why?

Because:

```dockerfile
COPY . .
```

copies the available contents of the build context.

---

### Step 2 — Create `.dockerignore`

Create:

```text
.dockerignore
```

Add:

```text
secret.txt
*.log
```

Your project now looks like:

```text
dockerignore-demo/
├── Dockerfile
├── .dockerignore
├── app.py
├── secret.txt
└── notes.log
```

Notice:

**`secret.txt` and `notes.log` still exist on your host.**

`.dockerignore` has not deleted anything.

It simply prevents those files from being included in the build context.

---

### Step 3 — Rebuild

Build a new image:

```bash
docker build --no-cache -t ignore-demo:2.0 .
```

Inspect the image:

```bash
docker run --rm --entrypoint ls ignore-demo:2.0 -la /app
```

Now you should see:

```text
app.py
```

The excluded files should not be there.

Finally, verify the application still works:

```bash
docker run --rm ignore-demo:2.0
```

Expected:

```text
Hello from Docker!
```

---

## 10. Verify the Build Context

During the build, Docker normally shows a build-context loading step.

For example, with:

```bash
docker build --progress=plain -t ignore-demo:3.0 .
```

you may see something similar to:

```text
[internal] load build context
transferring context: ...
```

The exact output depends on the Docker version and builder.

After adding `.dockerignore`, the amount of context may become smaller because excluded files are no longer part of the context.

However, don't treat the displayed byte count as a complete inventory of which individual files were included.

For this exercise, the most useful verification is:

```bash
docker run --rm --entrypoint ls ignore-demo:2.0 -la /app
```

That confirms what `COPY . .` actually placed into the image.

---

## 11. Important Things to Remember

Keep these distinctions clear:

| Concept            | Meaning                                                               |
| ------------------ | --------------------------------------------------------------------- |
| Build context      | Files available to the Docker build                                   |
| `.dockerignore`    | Filters files out of the build context                                |
| `COPY`             | Copies files from the available context into the image                |
| `.gitignore`       | Controls what Git normally ignores                                    |
| `*.log`            | Matches `.log` files                                                  |
| `**/*.log`         | Matches `.log` files through nested directories                       |
| `!pattern`         | Re-includes a previously excluded match, subject to the pattern rules |
| `.dockerignore`    | Does not delete files from your host                                  |
| Image optimization | Not the same thing as reducing the build context                      |

One important point:

**`.dockerignore` does not automatically make every Docker image smaller.**

It can reduce the image size when excluded files would otherwise have been copied into the image.

But it cannot fix things such as:

```dockerfile
RUN apt install unnecessary-package
```

or:

```dockerfile
FROM a-large-base-image
```

Those are image-optimization concerns.

We'll address those in Phase 4.


---

**Phase 4 — Docker Image Optimization**

where we'll start looking at how to make the **image itself** smaller, cleaner, and more production-oriented.
