# Phase 1 — Topic 2: Dockerfile

We are continuing **strictly in the order of the Docker Image Building Guide**.

```text
Phase 1
│
├── 1. Container vs Image       ✅
├── 2. Dockerfile               ← NOW
├── 3. Build context
├── 4. Docker image layers
├── 5. Docker image registry
├── 6. Container filesystem
├── 7. Base images
└── 8. Image tags
```

The guide places **Dockerfile immediately after Container vs Image**, and later introduces the core Dockerfile instructions during the first image-building phase. 

---

# 2. What is a Dockerfile?

A **Dockerfile** is a text file that contains instructions describing **how Docker should build an image**.

Think of it as a **recipe for creating a Docker image**.

The relationship is:

```text
Dockerfile
    │
    │ describes how to build
    ▼
Docker Image
    │
    │ used to create
    ▼
Container
```

So:

> **Dockerfile = instructions for building an image**

It is **not** the image itself.

It is also **not** the container.

---

# 2.1 Simple example

Suppose you have:

```text
my-app/
├── app.py
└── Dockerfile
```

Your Dockerfile might eventually contain:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

This tells Docker, conceptually:

```text
Start with Python
      ↓
Create/use /app
      ↓
Put app.py into /app
      ↓
Run python app.py when container starts
```

Then:

```bash
docker build -t my-app:1.0 .
```

uses that Dockerfile to build:

```text
my-app:1.0
```

which is the **image**.

---

# 2.2 Dockerfile is declarative

An important concept is that a Dockerfile generally describes **what the resulting image should contain and how it should behave**.

For example:

```dockerfile
FROM python:3.12
WORKDIR /app
COPY app.py .
CMD ["python", "app.py"]
```

You're describing the desired image configuration.

You're not manually creating the image filesystem yourself.

Docker's build system reads those instructions and performs the required build operations.

Conceptually:

```text
              Dockerfile
                  │
                  ▼
        Docker build engine
                  │
       interprets instructions
                  │
                  ▼
             Docker Image
```

---

# 2.3 Why do we need Dockerfile?

Without a Dockerfile, you could manually create containers and install things inside them.

For example, conceptually:

```text
Start container
      ↓
Install Python
      ↓
Copy application
      ↓
Install dependencies
      ↓
Configure application
      ↓
Run application
```

But this is problematic.

If you need to recreate the environment tomorrow, you have to remember all those steps.

And if another engineer needs the same environment, they need to reproduce the same process.

A Dockerfile turns those build instructions into something that can be **stored with your source code and reproduced**.

```text
Dockerfile
    │
    ├── version controlled
    ├── repeatable
    ├── reviewable
    └── usable by CI/CD
```

That's particularly important for your CI/CD goal.

---

# 2.4 Dockerfile as a build instruction set

Consider:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

Don't worry yet about understanding every instruction.

We'll study them later.

For now, just understand that each line represents an instruction involved in building or configuring the image.

Conceptually:

```text
FROM
  ↓
WORKDIR
  ↓
COPY
  ↓
RUN
  ↓
COPY
  ↓
CMD
```

Docker processes these instructions as part of the image build.

---

# 2.5 Build instructions vs runtime instructions

This distinction is extremely important.

Some Dockerfile instructions affect **image building**.

For example:

```dockerfile
RUN pip install -r requirements.txt
```

This command executes during the **image build**.

Other instructions describe what should happen when a container is started.

For example:

```dockerfile
CMD ["python", "app.py"]
```

This describes the default command for the **container runtime**.

So conceptually:

```text
                 Dockerfile
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
     Build behavior        Runtime behavior
          │                     │
          ▼                     ▼
       RUN, COPY             CMD, ENTRYPOINT
          │
          ▼
      Docker Image
          │
          ▼
      Container
```

We will study the exact behavior of each instruction later.

---

# 2.6 The core instructions from your roadmap

Your project guide specifically identifies these instructions for the first image-building phase: 

```text
FROM
WORKDIR
COPY
RUN
ENV
EXPOSE
CMD
ENTRYPOINT
```

Don't memorize them yet.

First understand what category they belong to.

| Instruction  | Basic purpose                                         |
| ------------ | ----------------------------------------------------- |
| `FROM`       | Select the starting/base image                        |
| `WORKDIR`    | Set the working directory                             |
| `COPY`       | Copy files into the image                             |
| `RUN`        | Execute a command during build                        |
| `ENV`        | Define environment variables                          |
| `EXPOSE`     | Declare an intended container port                    |
| `CMD`        | Define the default container command                  |
| `ENTRYPOINT` | Define the container's main executable/entry behavior |

We'll eventually go through these **one by one**.

---

# 2.7 Dockerfile → Image

This is the most important relationship for this topic.

Suppose we have:

```text
my-app/
├── app.py
└── Dockerfile
```

We run:

```bash
docker build -t my-app:1.0 .
```

The simplified flow is:

```text
        Dockerfile
            +
       application files
            │
            │ docker build
            ▼
      Docker build process
            │
            ▼
      ┌──────────────┐
      │ Docker Image │
      │  my-app:1.0  │
      └──────────────┘
```

Then:

```bash
docker run my-app:1.0
```

creates a container from that image:

```text
Dockerfile
    │
    │ docker build
    ▼
  Image
    │
    │ docker run
    ▼
Container
```

That complete relationship should now be clear.

---

# 2.8 Dockerfile is normally stored with application source

A common project structure is:

```text
my-app/
├── app.py
├── requirements.txt
├── Dockerfile
└── ...
```

This is useful because your Git repository can contain both:

```text
Application source
        +
Docker build instructions
```

Then CI can check out the repository and build the image.

Eventually your CI/CD flow will look like:

```text
Git repository
      │
      ├── application source
      │
      └── Dockerfile
              │
              ▼
          CI pipeline
              │
              ▼
         docker build
              │
              ▼
         Docker Image
```

This is exactly why Dockerfiles become important once we reach the CI/CD portion of your roadmap.

---

# 2.9 Dockerfile vs Docker image

Don't mix these up.

```text
Dockerfile
    │
    │ instructions
    ▼
Docker Image
    │
    │ instance
    ▼
Container
```

### Dockerfile

Human-readable instructions.

Example:

```dockerfile
FROM python:3.12
WORKDIR /app
COPY app.py .
CMD ["python", "app.py"]
```

### Image

The built artifact produced from those instructions.

Example:

```text
my-app:1.0
```

### Container

A running instance of that image.

Example:

```text
my-app-container
```

---

# 2.10 One very important CI/CD concept

Eventually, your CI server will execute something like:

```bash
docker build -t my-app:1.0 .
```

The CI server doesn't need to manually understand how to install your application's dependencies.

The **Dockerfile contains the build instructions**.

Therefore:

```text
Developer
    │
    │ pushes source + Dockerfile
    ▼
Git repository
    │
    ▼
CI server
    │
    │ docker build
    ▼
Docker Image
```

This makes the Dockerfile part of your application's **build definition**.

---

# 2.11 What we are NOT learning yet

To maintain the roadmap strictly, we're **not going deep into**:

* Build context
* Image layers
* Build cache
* `.dockerignore`
* Multi-stage builds
* Image optimization
* Registry
* Tags
* BuildKit

Those are later topics.

For example, although you will see `.` here:

```bash
docker build -t my-app:1.0 .
```

we are **not yet explaining what that `.` means**.

That's the next roadmap topic:

> **Build context**

This keeps our learning progression clean.

---

# Phase 1 progress

```text
01. Container vs Image       ✅
02. Dockerfile               ✅
03. Build context            ← NEXT
04. Docker image layers
05. Docker image registry
06. Container filesystem
07. Base images
08. Image tags
```

### The mental model you should have now

```text
                 Dockerfile
              "How to build it"
                     │
                     │
                     ▼
              Docker Image
              "What we build"
                     │
                     │
                     ▼
                Container
                "What we run"
```

And the two commands we'll repeatedly encounter are:

```bash
docker build
```

→ **builds the image**

```bash
docker run
```

→ **creates/runs a container from the image**

The **next topic, strictly according to the guide, is Build Context**.
