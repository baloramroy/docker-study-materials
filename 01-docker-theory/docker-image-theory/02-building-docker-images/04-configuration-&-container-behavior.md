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

Let's start with:

```dockerfile
ENV APP_NAME="my-app"
```

`ENV` defines an **environment variable** in the image.

Conceptually:

```text
Docker Image
    │
    └── Environment
          │
          └── APP_NAME=my-app
```

When a container is created from that image, the variable is available inside the container.

For example, a process inside the container can access:

```text
APP_NAME=my-app
```

---

# 2. Why Do We Need Environment Variables?

Applications often need configuration that shouldn't be hardcoded directly into the application.

For example:

```text
APP_ENV
LOG_LEVEL
DATABASE_HOST
DATABASE_PORT
API_URL
```

Instead of putting configuration directly into application code:

```python
database_host = "10.10.10.50"
```

the application can read:

```text
DATABASE_HOST
```

from its environment.

This allows the same image to be used in different environments.

For example:

```text
Same Image
    │
    ├── Development
    │     DATABASE_HOST=dev-db
    │
    ├── Testing
    │     DATABASE_HOST=test-db
    │
    └── Production
          DATABASE_HOST=prod-db
```

The application image doesn't necessarily need to change just because its runtime configuration changes.

---

# 3. Using `ENV` in Our Dockerfile

We could add:

```dockerfile
ENV APP_NAME="my-app"
```

Our Dockerfile would become:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY app.py .

ENV APP_NAME="my-app"

CMD ["python", "app.py"]
```

Now the resulting image contains the environment variable:

```text
APP_NAME=my-app
```

---

# 4. `ENV` Does Not Execute Anything

This is worth separating from `RUN`.

Compare:

```dockerfile
RUN pip install flask
```

with:

```dockerfile
ENV APP_NAME="my-app"
```

`RUN`:

```text
Execute a command
        ↓
During build
```

`ENV`:

```text
Define configuration
        ↓
Available in the container environment
```

So:

```text
RUN → perform an action

ENV → define an environment variable
```

---

# 5. Important: Don't Put Secrets in the Dockerfile

There's an important practical rule here.

Avoid putting sensitive credentials directly into:

```dockerfile
ENV DB_PASSWORD="my-secret-password"
```

Why?

Because the Dockerfile itself contains the secret, and the resulting image configuration can also expose it.

So things like:

```text
Passwords
API keys
Access tokens
Private credentials
```

should generally **not be baked into the image** this way.

A better approach is to provide sensitive configuration at runtime using an appropriate secret-management mechanism.

We won't go deeply into Docker secrets or external secret managers yet.

For now, remember:

> **`ENV` is suitable for configuration, but don't treat the Dockerfile as a secure place to store secrets.**

---

# 6. `EXPOSE` — Declare an Intended Container Port

Now consider:

```dockerfile
EXPOSE 8080
```

`EXPOSE` tells Docker:

> **This container is intended to listen on port `8080`.**

For example, suppose our application is a web server:

```text
Application
    │
    ▼
Listening on
TCP 8080
```

We could document that in the Dockerfile:

```dockerfile
EXPOSE 8080
```

---

# 7. Does `EXPOSE` Publish the Port?

**No.**

This is one of the most important things to understand about `EXPOSE`.

If we write:

```dockerfile
EXPOSE 8080
```

it does **not** automatically create:

```text
Host:8080
     │
     ▼
Container:8080
```

Instead, it records the intended container port as image metadata/documentation.

Actual host-to-container port publishing is done when the container is started.

For example:

```bash
docker run -p 8080:8080 my-app:1.0
```

The `-p` option is what establishes the port mapping.

---

# 8. `EXPOSE` vs `-p`

Keep these separate:

### Dockerfile

```dockerfile
EXPOSE 8080
```

means:

> The application is expected to use port 8080 inside the container.

### Runtime

```bash
docker run -p 8080:8080 my-app:1.0
```

means:

> Publish host port 8080 and map it to container port 8080.

Conceptually:

```text
Dockerfile
EXPOSE 8080
      │
      ▼
Documentation / metadata


docker run
-p 8080:8080
      │
      ▼
Actual port publishing
```

This distinction will become much more important when we containerize a web application.

---

# 9. Our Current Application Doesn't Need `EXPOSE`

Our application is:

```python
print("Hello from Docker!")
```

It doesn't listen on a network port.

Therefore:

```dockerfile
EXPOSE 8080
```

wouldn't make sense for our current example.

That's intentional.

We are learning the instruction without adding unnecessary configuration to the application.

Later, when we build a web application, `EXPOSE` will have a practical purpose.

---

# 10. `ENTRYPOINT` — Define the Main Executable

Now we reach the instruction that requires a little more thought:

```dockerfile
ENTRYPOINT
```

`ENTRYPOINT` defines the **main executable** for the container.

For example:

```dockerfile
ENTRYPOINT ["python"]
```

This tells Docker:

> The main executable for this container is `python`.

Then we can pair it with:

```dockerfile
CMD ["app.py"]
```

Together:

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

produce the effective command:

```bash
python app.py
```

---

# 11. Why Have Both `ENTRYPOINT` and `CMD`?

At first, this can seem unnecessary.

Why not simply write:

```dockerfile
CMD ["python", "app.py"]
```

That's perfectly valid for our current application.

The distinction becomes useful when we want to separate:

```text
What executable should always be used?
```

from:

```text
What default arguments should it receive?
```

For example:

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

can be thought of as:

```text
ENTRYPOINT
    │
    ▼
  python
    +
CMD
    │
    ▼
  app.py
    │
    ▼
python app.py
```

---

# 12. `CMD` vs `ENTRYPOINT`

Let's establish a simple mental model.

### `ENTRYPOINT`

Defines the main executable.

```dockerfile
ENTRYPOINT ["python"]
```

Think:

> **What program is this container fundamentally running?**

### `CMD`

Provides the default command/arguments.

```dockerfile
CMD ["app.py"]
```

Think:

> **What should that executable run by default?**

Together:

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

becomes:

```bash
python app.py
```

---

# 13. Why Not Always Use `ENTRYPOINT`?

Because `CMD` by itself is often simpler when the container's startup command is straightforward.

Our original:

```dockerfile
CMD ["python", "app.py"]
```

is perfectly reasonable.

We don't need to turn every simple Dockerfile into:

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

just because `ENTRYPOINT` exists.

The choice depends on the behavior we want.

For our first image, `CMD` is easier to understand.

---

# 14. A Practical Comparison

Consider these two Dockerfiles.

### Option A

```dockerfile
CMD ["python", "app.py"]
```

This says:

> The default command is `python app.py`.

### Option B

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

This says:

> The main executable is `python`, and the default argument is `app.py`.

Both result in the application starting as:

```bash
python app.py
```

But they express the container's behavior differently.

---

# 15. Why `ENTRYPOINT` Becomes Useful

Imagine we build a container intended to behave like a specific command.

For example:

```dockerfile
ENTRYPOINT ["python"]
```

Then different default arguments could be supplied:

```text
python app.py
python script.py
python worker.py
```

The executable remains:

```text
python
```

while the arguments can vary.

This is one reason `ENTRYPOINT` is useful when designing images around a specific executable.

We will revisit command overriding more deeply when we practice running containers.

---

# 16. A Small Warning About `CMD` and `ENTRYPOINT`

There are several rules governing how:

```text
CMD
ENTRYPOINT
docker run
```

interact.

For example, a command supplied after the image name can replace `CMD` in certain configurations, while `ENTRYPOINT` changes how that command is interpreted.

Those details matter in real Docker usage, but **we don't need to memorize every combination right now**.

For this phase, keep the core model:

```text
CMD
→ default command / arguments

ENTRYPOINT
→ main executable
```

We'll reinforce the behavior through actual commands in Part 5.

---

# 17. Our Dockerfile — Two Valid Styles

Our current application can use the simple form:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

Or we could express the startup behavior as:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY app.py .

ENTRYPOINT ["python"]
CMD ["app.py"]
```

For **our first Docker image**, we'll keep the first version.

Why?

Because our goal is to learn Docker's concepts progressively, not add complexity where we don't need it.

---

# 18. Where Do `ENV` and `EXPOSE` Fit?

Let's put all the instructions we've learned into one conceptual model:

```text
FROM
 ↓
Starting environment

WORKDIR
 ↓
Application working directory

COPY
 ↓
Application files

RUN
 ↓
Build-time operations

ENV
 ↓
Environment configuration

EXPOSE
 ↓
Intended container port

ENTRYPOINT
 ↓
Main executable

CMD
 ↓
Default runtime command/arguments
```

Notice how the instructions have different jobs.

---

# 19. The Three Categories We've Now Learned

We can simplify the Dockerfile instructions into three broad groups.

### 1. Establish the image

```text
FROM
WORKDIR
COPY
```

These help construct the image environment and application filesystem.

### 2. Modify/configure during build

```text
RUN
ENV
```

`RUN` performs build-time operations, while `ENV` establishes environment configuration.

### 3. Define runtime behavior

```text
EXPOSE
ENTRYPOINT
CMD
```

These describe or configure how the resulting container is intended to behave.

This isn't a strict technical classification of every Dockerfile instruction, but it's a useful mental model for where we are now.

---

# 20. Our Full Dockerfile — Understanding Every Line

Let's return to our simple Dockerfile:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

We can now explain every line:

```text
FROM python:3.12
    ↓
Start with the Python 3.12 image.

WORKDIR /app
    ↓
Use /app as the working directory.

COPY app.py .
    ↓
Copy our application into /app.

CMD ["python", "app.py"]
    ↓
Run python app.py by default when the container starts.
```

And we understand the instructions we didn't use:

```text
RUN
→ Would execute something during build.

ENV
→ Would define an environment variable.

EXPOSE
→ Would document an intended container port.

ENTRYPOINT
→ Could define the main executable.
```

---

# 21. Part 4 Checkpoint

Before moving to the actual build, make sure these distinctions are clear.

### `ENV`

```dockerfile
ENV APP_NAME="my-app"
```

Defines an environment variable.

---

### `EXPOSE`

```dockerfile
EXPOSE 8080
```

Documents an intended container port.

It does **not** publish that port to the host.

---

### `ENTRYPOINT`

```dockerfile
ENTRYPOINT ["python"]
```

Defines the main executable.

---

### `CMD`

```dockerfile
CMD ["app.py"]
```

Provides the default command/arguments.

Together:

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

effectively means:

```bash
python app.py
```

---

# The Mental Model

At this point, you should be able to look at a Dockerfile and mentally follow its purpose:

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

And the most important distinctions from Parts 2–4 are:

```text
FROM
→ Where the image starts

WORKDIR
→ Where we work

COPY
→ What files we add

RUN
→ What happens during build

ENV
→ What environment configuration exists

EXPOSE
→ What container port is intended

ENTRYPOINT
→ What executable the container fundamentally runs

CMD
→ What it runs by default
```

**Part 4 is complete.**

We now understand the core Dockerfile instructions needed for our first image.

In **Part 5 — Assemble, Build & Run**, we'll stop learning individual instructions and actually put everything together:

```text
Dockerfile
    ↓
docker build -t my-app:1.0 .
    ↓
Docker Image
    ↓
docker images
    ↓
docker image inspect my-app:1.0
    ↓
docker run my-app:1.0
    ↓
Hello from Docker!
```

That will be the point where everything we've learned so far becomes a real Docker image.
