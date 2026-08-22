# Phase 2 — Part 2: `ENV`, `EXPOSE`, and `ENTRYPOINT`

We continue **exactly where we stopped**.

## Phase 2 progress

```text
01. Create simple application       ✅
02. Understand Dockerfile           ✅
03. FROM                            ✅
04. WORKDIR                         ✅
05. COPY                            ✅
06. RUN                             ✅
07. CMD                             ✅

08. ENV                             ← NOW
09. EXPOSE
10. ENTRYPOINT
11. docker build
12. Inspect image
13. Run container
```

Our focus in this lesson is:

```text
ENV
EXPOSE
ENTRYPOINT
```

And, most importantly, we'll understand how they differ from:

```text
RUN
CMD
```

---

# 1. First: why do we need these instructions?

So far our Dockerfile looks like:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY app.py .

CMD ["python", "app.py"]
```

This is already enough to build and run a simple application.

But real applications usually need additional information.

For example:

```text
Application
    │
    ├── needs environment variables
    ├── listens on a port
    └── needs a specific startup command
```

Docker provides different instructions for these different purposes.

---

# 2. `ENV` — Environment Variables

Let's start with:

```dockerfile
ENV
```

`ENV` allows us to define **environment variables inside the image**.

For example:

```dockerfile
ENV APP_ENV=production
```

This creates an environment variable:

```text
APP_ENV=production
```

inside the container.

---

# 3. Why are environment variables useful?

Suppose our Python application does this:

```python
import os

environment = os.getenv("APP_ENV")

print(f"Running in {environment}")
```

If our Dockerfile contains:

```dockerfile
ENV APP_ENV=production
```

then when the container runs:

```bash
docker run my-app:1.0
```

the application can read:

```text
APP_ENV=production
```

and produce:

```text
Running in production
```

So the flow is:

```text
Dockerfile
    │
    │ ENV APP_ENV=production
    ▼
Image
    │
    │ docker run
    ▼
Container
    │
    ▼
Environment variable
    │
    ▼
Application
```

---

# 4. `ENV` isn't only for the application

An environment variable is simply a variable available inside the container environment.

For example:

```dockerfile
ENV APP_HOME=/app
ENV APP_ENV=production
```

Then inside the container:

```text
APP_HOME=/app
APP_ENV=production
```

You can verify environment variables with:

```bash
docker run my-app:1.0 env
```

You'll see many environment variables, including the ones you defined.

---

# 5. `ENV` can also be overridden

This is important.

Suppose Dockerfile contains:

```dockerfile
ENV APP_ENV=production
```

But you run:

```bash
docker run -e APP_ENV=development my-app:1.0
```

Now the container gets:

```text
APP_ENV=development
```

rather than:

```text
APP_ENV=production
```

So:

```text
Dockerfile default
       │
       │ ENV APP_ENV=production
       ▼
   Container
       │
       │ docker run -e APP_ENV=development
       ▼
APP_ENV=development
```

This is extremely useful in CI/CD because the **same image** can be used in different environments.

For example:

```text
Same Image
    │
    ├── Development → APP_ENV=development
    │
    ├── Staging     → APP_ENV=staging
    │
    └── Production  → APP_ENV=production
```

The image doesn't necessarily need to be rebuilt for each environment.

---

# 6. Very important: don't put secrets into `ENV`

You might see something like:

```dockerfile
ENV DB_PASSWORD=MySecretPassword
```

Technically Docker allows this.

But it is a **bad practice**.

Why?

Because the value becomes part of the image configuration.

A Docker image should generally not contain things like:

```text
database passwords
API keys
private keys
access tokens
```

Later, when we study image security and CI/CD secrets, we'll cover proper approaches.

For now, remember:

> **`ENV` is for configuration, not secrets.**

---

# 7. `EXPOSE` — What does it actually do?

Now:

```dockerfile
EXPOSE
```

Suppose our application is a web application listening on port `8080`.

We can write:

```dockerfile
EXPOSE 8080
```

At first glance, people often think this means:

> "Docker opens port 8080."

That's not quite correct.

---

# 8. What `EXPOSE` actually means

`EXPOSE` is primarily **metadata/documentation** saying:

> "This containerized application is expected to listen on this port."

For example:

```dockerfile
EXPOSE 8080
```

communicates:

```text
Application
    │
    ▼
Listening inside container
       port 8080
```

But it does **not automatically publish that port to the host**.

This distinction is extremely important.

---

# 9. Example

Suppose our application listens on:

```text
8080
```

Dockerfile:

```dockerfile
EXPOSE 8080
```

We build:

```bash
docker build -t my-app:1.0 .
```

Then:

```bash
docker run my-app:1.0
```

The container may have the application listening on:

```text
container:8080
```

But your host does not automatically get:

```text
host:8080
```

---

# 10. Publishing the port

To publish it, we use:

```bash
docker run -p 8080:8080 my-app:1.0
```

This means:

```text
HOST PORT : CONTAINER PORT
     │             │
     ▼             ▼
    8080          8080
```

Conceptually:

```text
Host
┌───────────────┐
│    :8080      │
└───────┬───────┘
        │
        │ -p 8080:8080
        ▼
┌───────────────┐
│   Container   │
│               │
│    :8080      │
│       │       │
│       ▼       │
│   Application │
└───────────────┘
```

So remember:

> **`EXPOSE` describes the container port.**

> **`-p` publishes/maps the container port to the host.**

---

# 11. `EXPOSE` vs `-p`

This distinction is worth memorizing.

| Feature                          | `EXPOSE` | `-p` |
| -------------------------------- | -------- | ---- |
| Declares application port        | Yes      | No   |
| Publishes host port              | No       | Yes  |
| Dockerfile instruction           | Yes      | No   |
| Used with `docker run`           | No       | Yes  |
| Creates host → container mapping | No       | Yes  |

Example:

```dockerfile
EXPOSE 8080
```

and:

```bash
docker run -p 8080:8080 my-app:1.0
```

These serve **different purposes**.

---

# 12. `EXPOSE` does not mean the application actually listens

Another important point.

If your Dockerfile says:

```dockerfile
EXPOSE 8080
```

but your application actually listens on:

```text
5000
```

Docker isn't going to magically move it to port `8080`.

The application itself determines where it listens.

Think:

```text
EXPOSE
  │
  └── documentation/metadata

Application
  │
  └── actually listens on a port
```

---

# 13. Now `ENTRYPOINT`

This is the most important part of today's lesson.

We already know:

```dockerfile
CMD
```

For example:

```dockerfile
CMD ["python", "app.py"]
```

Now we introduce:

```dockerfile
ENTRYPOINT
```

For example:

```dockerfile
ENTRYPOINT ["python", "app.py"]
```

At first they look almost identical.

That's why they're frequently confusing.

---

# 14. What does `ENTRYPOINT` mean?

`ENTRYPOINT` defines the **main executable/process** of the container.

For example:

```dockerfile
ENTRYPOINT ["python", "app.py"]
```

means:

> When this container starts, its primary process is `python app.py`.

Conceptually:

```text
docker run
    │
    ▼
ENTRYPOINT
    │
    ▼
python app.py
    │
    ▼
Application
```

---

# 15. So why do we have both `CMD` and `ENTRYPOINT`?

Because they serve slightly different purposes.

Think about this:

```text
ENTRYPOINT = What the container fundamentally runs

CMD        = Default arguments/default command behavior
```

A very useful pattern is:

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

Now Docker effectively runs:

```bash
python app.py
```

---

# 16. Why is this useful?

Suppose:

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

Running:

```bash
docker run my-app:1.0
```

results in:

```text
python app.py
```

But if we run:

```bash
docker run my-app:1.0 another.py
```

the command becomes:

```text
python another.py
```

The `ENTRYPOINT` remains:

```text
python
```

while the default `CMD` can be replaced.

This is the key relationship.

---

# 17. `CMD` can be overridden

Suppose:

```dockerfile
CMD ["python", "app.py"]
```

Then:

```bash
docker run my-app:1.0
```

runs:

```text
python app.py
```

But:

```bash
docker run my-app:1.0 python another.py
```

can replace the default command.

So `CMD` is relatively flexible.

---

# 18. `ENTRYPOINT` behaves differently

Suppose:

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

Running:

```bash
docker run my-app:1.0
```

gives:

```text
python app.py
```

Running:

```bash
docker run my-app:1.0 another.py
```

gives:

```text
python another.py
```

The arguments supplied to `docker run` are appended to the `ENTRYPOINT`.

That's one of the main reasons to combine them.

---

# 19. The easiest mental model

Think of:

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

as:

```text
ENTRYPOINT
    +
CMD
    =
Final command
```

So:

```text
python + app.py
      ↓
python app.py
```

If the user supplies:

```bash
docker run my-app:1.0 test.py
```

then:

```text
python + test.py
      ↓
python test.py
```

---

# 20. `RUN`, `CMD`, and `ENTRYPOINT`

Now we can compare the three.

| Instruction  | Purpose                                   | Happens           |
| ------------ | ----------------------------------------- | ----------------- |
| `RUN`        | Execute a command to build the image      | Build time        |
| `CMD`        | Provide default startup command/arguments | Container runtime |
| `ENTRYPOINT` | Define the main executable                | Container runtime |

Example:

```dockerfile
RUN pip install flask
```

means:

```text
During docker build:
    install Flask
```

Then:

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

means:

```text
When container starts:
    python app.py
```

---

# 21. A complete Dockerfile

Let's now put today's concepts together.

```dockerfile
FROM python:3.12

WORKDIR /app

ENV APP_ENV=production

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY app.py .

EXPOSE 8080

ENTRYPOINT ["python"]

CMD ["app.py"]
```

Now each major instruction has a clear responsibility:

```text
FROM
 ↓
Choose Python environment

WORKDIR
 ↓
Set /app

ENV
 ↓
Set application configuration

COPY
 ↓
Bring source files into image

RUN
 ↓
Install dependencies during build

EXPOSE
 ↓
Declare application port

ENTRYPOINT
 ↓
Define main executable

CMD
 ↓
Provide default argument
```

---

# 22. One subtle but important correction

You should **not** think of Dockerfile instructions as all doing the same kind of thing.

They fall into different conceptual categories.

### Image construction

```text
FROM
WORKDIR
COPY
RUN
```

These help construct the image.

### Image configuration / metadata

```text
ENV
EXPOSE
```

These configure or describe the resulting image/container environment.

### Container startup

```text
ENTRYPOINT
CMD
```

These determine what happens when the container starts.

This classification will help you understand Dockerfiles instead of memorizing syntax.

---

# 23. The complete build → run lifecycle

Now let's connect everything we've learned.

### Step 1 — Source

```text
my-app/
├── app.py
├── requirements.txt
└── Dockerfile
```

### Step 2 — Build

```bash
docker build -t my-app:1.0 .
```

Docker processes:

```text
FROM
 ↓
WORKDIR
 ↓
ENV
 ↓
COPY
 ↓
RUN
 ↓
COPY
 ↓
EXPOSE
 ↓
ENTRYPOINT
 ↓
CMD
```

and produces:

```text
my-app:1.0
```

### Step 3 — Run

```bash
docker run -p 8080:8080 my-app:1.0
```

Now Docker starts the container.

The runtime configuration gives us:

```text
ENTRYPOINT = python
CMD        = app.py
```

Therefore:

```text
python app.py
```

runs inside the container.

---

# 24. The five instructions you should now clearly distinguish

This is the key takeaway from this lesson.

### `RUN`

```dockerfile
RUN pip install flask
```

**Build the image.**

---

### `ENV`

```dockerfile
ENV APP_ENV=production
```

**Provide environment configuration.**

---

### `EXPOSE`

```dockerfile
EXPOSE 8080
```

**Declare the port the application is expected to use.**

It does not publish the port.

---

### `ENTRYPOINT`

```dockerfile
ENTRYPOINT ["python"]
```

**Define the main executable.**

---

### `CMD`

```dockerfile
CMD ["app.py"]
```

**Provide the default command/arguments.**

---

# 25. A practical mental model

When you see this:

```dockerfile
FROM python:3.12

WORKDIR /app

ENV APP_ENV=production

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY app.py .

EXPOSE 8080

ENTRYPOINT ["python"]

CMD ["app.py"]
```

you should be able to mentally translate it to:

```text
Start with Python
       ↓
Work inside /app
       ↓
Set application environment
       ↓
Copy dependency definition
       ↓
Install dependencies
       ↓
Copy application
       ↓
Application expects port 8080
       ↓
Run Python
       ↓
Default Python argument is app.py
```

That's the understanding we're aiming for.

---

# Phase 2 Progress

```text
Phase 2 — Build Your First Image

01. Create simple application       ✅
02. Understand Dockerfile           ✅
03. FROM                            ✅
04. WORKDIR                         ✅
05. COPY                            ✅
06. RUN                             ✅
07. CMD                             ✅
08. ENV                             ✅
09. EXPOSE                          ✅
10. ENTRYPOINT                      ✅

11. docker build                    ← NEXT
12. Inspect image
13. Run container
```

Next we'll do something important: **actually build the image and inspect what Docker created**.

We'll use commands such as:

```bash
docker build
docker images
docker image inspect
docker history
docker run
```

and connect the command output back to the Dockerfile instructions we've just learned.
