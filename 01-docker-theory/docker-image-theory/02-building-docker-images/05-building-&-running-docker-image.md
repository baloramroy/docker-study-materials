# Phase 2 — Build Your First Docker Image

## Part 4 — Configuration & Container Behavior

### Where We Are

In Part 3, we established the most important timing distinction in a Dockerfile:

```text
RUN  → BUILD TIME
CMD  → RUNTIME
```

Our Dockerfile is now:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

We understand how the image gets its starting environment and application files, and we know how to define its default startup command.

Now we'll finish the remaining core instructions:

```text
ENV
EXPOSE
ENTRYPOINT
```

and then compare:

```text
CMD vs ENTRYPOINT
```

The focus here is no longer primarily **"what files go into the image?"**

Instead, we're looking at:

> **How do we configure the environment and define container behavior?**

---

# 1. `ENV` — Define Environment Variables

```dockerfile
ENV APP_NAME="my-app"
```

`ENV` defines an **environment variable** in the image.

```text
Docker Image
    │
    └── Environment
          │
          └── APP_NAME=my-app
```

When a container is created from that image, the variable is available inside it — a running process can read `APP_NAME=my-app` directly.

---

# 2. Why Do We Need Environment Variables?

Applications often need configuration that shouldn't be hardcoded into the code itself — things like:

```text
APP_ENV
LOG_LEVEL
DATABASE_HOST
DATABASE_PORT
API_URL
```

Instead of hardcoding:

```python
database_host = "10.10.10.50"
```

the application reads `DATABASE_HOST` from its environment. This means the **same image** can be reused across environments just by changing what's injected:

```text
Same Image
    │
    ├── Development   → DATABASE_HOST=dev-db
    ├── Testing       → DATABASE_HOST=test-db
    └── Production    → DATABASE_HOST=prod-db
```

The image doesn't need to change just because runtime configuration changes.

---

# 3. `ENV` Does Not Execute Anything

Compare:

```dockerfile
RUN pip install flask
```

with:

```dockerfile
ENV APP_NAME="my-app"
```

```text
RUN → perform an action, during build

ENV → define a value, available at runtime
```

If we added `ENV APP_NAME="my-app"` to our Dockerfile, it would sit between `COPY` and `CMD` — but our tiny app doesn't read any environment variables yet, so we won't add it to our working example. The concept is what matters here.

---

# 4. Important: Don't Put Secrets in the Dockerfile

Avoid putting sensitive credentials directly into `ENV`:

```dockerfile
ENV DB_PASSWORD="my-secret-password"
```

**Why this matters:** the Dockerfile itself contains the secret in plain text, and that value becomes part of the image's configuration/history — anyone who can pull or inspect the image can potentially see it. Passwords, API keys, access tokens, and other credentials should generally not be baked into an image this way.

A better approach is to supply sensitive configuration at runtime through a proper secret-management mechanism. We won't go deep into that yet — the rule to remember for now is:

> **`ENV` is for configuration, not for secrets.**

---

# 5. `EXPOSE` — Declare an Intended Container Port

```dockerfile
EXPOSE 8080
```

`EXPOSE` tells Docker: **this container is intended to listen on port 8080.** If our application were a web server, we'd document that intent here.

**Important:** `EXPOSE` does **not** publish the port. Writing `EXPOSE 8080` does not create a `Host:8080 → Container:8080` mapping. It's documentation/metadata, nothing more.

Actual port publishing happens at `docker run` time:

```bash
docker run -p 8080:8080 my-app:1.0
```

```text
EXPOSE 8080          →  documentation, at build time
docker run -p 8080:8080  →  actual publishing, at run time
```

Our current application doesn't listen on a network port, so we won't add `EXPOSE` to our Dockerfile yet — but this distinction will matter a lot once we containerize a web app.

---

# 6. `ENTRYPOINT` — Define the Main Executable

```dockerfile
ENTRYPOINT ["python"]
```

`ENTRYPOINT` defines the container's **main executable** — the program this container fundamentally runs. Paired with `CMD`:

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

Docker combines them into one effective command:

```bash
python app.py
```

A simple way to hold the two apart:

```text
ENTRYPOINT → what program does this container run?
CMD        → what should it run by default?
```

---

# 7. Why Have Both, Instead of Just `CMD`?

For our current app, `CMD ["python", "app.py"]` alone is perfectly fine — we don't need `ENTRYPOINT` just because it exists.

`ENTRYPOINT` earns its place when you want the **executable fixed** while the **arguments vary**. For example:

```dockerfile
ENTRYPOINT ["python"]
```

lets different containers built from a similar pattern supply different scripts as arguments:

```text
python app.py
python script.py
python worker.py
```

The executable (`python`) stays constant; only what it runs changes. That's the scenario `ENTRYPOINT` is designed for — and it's why production images built "around" a specific tool (like a CLI) often use it.

For our first image, we'll keep things simple:

```dockerfile
CMD ["python", "app.py"]
```

There are more detailed rules about how `CMD`, `ENTRYPOINT`, and arguments passed to `docker run` interact — we don't need to memorize them now. We'll see the behavior directly with real commands once we start running containers in Part 5.

---

# 8. The Three Categories We've Now Learned

We can group every instruction covered in Parts 2–4 into three jobs:

### Establish the image
```text
FROM      → starting environment
WORKDIR   → application directory
COPY      → application files
```

### Configure during build
```text
RUN       → build-time actions
ENV       → environment configuration
```

### Define runtime behavior
```text
EXPOSE      → intended container port
ENTRYPOINT  → main executable
CMD         → default command/arguments
```

This isn't a rigid technical classification — some instructions blur the lines — but it's a useful way to organize what each instruction is *for* as you read or write a Dockerfile.

---

# 9. Our Full Dockerfile — Understanding Every Line

```dockerfile
FROM python:3.12

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

```text
FROM python:3.12        → start with the Python 3.12 image
WORKDIR /app             → use /app as the working directory
COPY app.py .            → copy our application into /app
CMD ["python", "app.py"] → run python app.py by default at container start
```

And we understand what the unused instructions *would* do if we needed them:

```text
RUN         → execute something during build
ENV         → define an environment variable
EXPOSE      → document an intended container port
ENTRYPOINT  → define the container's main executable
```

---

# Part 4 Checkpoint

### `ENV`
Defines an environment variable available inside the container. Not for secrets.

### `EXPOSE`
Documents an intended container port. Does **not** publish it — `-p` at runtime does that.

### `ENTRYPOINT`
Defines the main executable.

### `CMD`
Provides the default command/arguments. Combined with `ENTRYPOINT ["python"]`, `CMD ["app.py"]` produces `python app.py`.

---

# The Mental Model

```text
                  Dockerfile
                      │
       ┌──────────────┼──────────────┐
       │              │              │
       ▼              ▼              ▼
     IMAGE        CONFIGURATION     RUNTIME
       │              │              │
   FROM/COPY         ENV          ENTRYPOINT
   WORKDIR           EXPOSE           CMD
       │
       ▼
   Docker Image
       │
       │ docker run
       ▼
   Container
       │
       ▼
 Main application
```

**Part 4 is complete.**

In **Part 5 — Assemble, Build & Run**, we stop learning individual instructions and put everything together:

```text
Dockerfile → docker build → Docker Image → docker images →
docker image inspect → docker run → Hello from Docker!
```