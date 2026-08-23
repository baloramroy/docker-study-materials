# Phase 1 — Part 2: Dockerfile

## Where we are

```text
Phase 1 — Docker Fundamentals

01. Container vs Image       ✅
02. Dockerfile               ← NOW
03. Build Context
04. Docker Image Layers
05. Docker Image Registry
06. Container Filesystem
07. Base Images
08. Image Tags
```

---

## 1. What is a Dockerfile?

A **Dockerfile is a text file containing instructions that Docker uses to build a Docker image.**

Example:

```dockerfile
FROM nginx
```

A Dockerfile is therefore a **build definition** for an image.

The important distinction is:

```text
Dockerfile
    │
    │ docker build
    ▼
Docker Image
    │
    │ docker run
    ▼
Docker Container
```

So:

* **Dockerfile** → instructions
* **Image** → built artifact/template
* **Container** → running instance of the image

This relationship should remain in our mental model throughout this project.

---

# 2. Why do we need a Dockerfile?

Suppose you have an application:

```text
my-app/
├── app.py
└── requirements.txt
```

You want to create an image containing that application.

Docker needs to know things such as:

* What environment should the image start from?
* Where should the application files go?
* What dependencies should be installed?
* What command should run when the container starts?

The Dockerfile defines those instructions.

For example:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

We're **not learning these instructions individually yet**. That will come later.

For now, understand that the Dockerfile describes how the image should be constructed.

---

# 3. Dockerfile vs Docker Image

These are different things.

### Dockerfile

A normal text file:

```text
Dockerfile
```

containing:

```dockerfile
FROM nginx
```

### Docker image

The result of processing that Dockerfile:

```text
my-nginx:1.0
```

The process is:

```text
Dockerfile
    │
    │ docker build
    ▼
Docker Image
```

This distinction is particularly important in CI/CD.

The **Dockerfile is part of your source/build definition**, while the **image is the build artifact**.

---

# 4. A Simple Project

Let's use this throughout our learning.

Create:

```text
docker-demo/
└── Dockerfile
```

Put this inside the Dockerfile:

```dockerfile
FROM nginx
```

Now build it:

```bash
docker build -t my-nginx:1.0 .
```

Conceptually:

```text
docker-demo/
    │
    └── Dockerfile
           │
           │ docker build
           ▼
      my-nginx:1.0
```

Check the image:

```bash
docker images
```

You should see something similar to:

```text
REPOSITORY   TAG   IMAGE ID
my-nginx     1.0   xxxxxxxxx
```

---

# 5. From Image to Container

Now use the image:

```bash
docker run -d --name my-nginx-container my-nginx:1.0
```

Check it:

```bash
docker ps
```

Now the complete process is:

```text
Dockerfile
    │
    │ docker build
    ▼
my-nginx:1.0
    │
    │ docker run
    ▼
my-nginx-container
```

This is the basic Docker workflow.

---

# 6. What Does `docker build` Mean?

Consider:

```bash
docker build -t my-nginx:1.0 .
```

There are three important parts:

### `docker build`

Tells Docker:

> Build an image.

### `-t my-nginx:1.0`

Gives the resulting image a name and tag.

We'll study tags properly in **Part 8**.

### `.`

Specifies the **build context**.

We'll study this in the **next part**.

So for now:

```text
docker build -t my-nginx:1.0 .
             │                │
             │                └── Build context
             └── Image name/tag
```

Don't worry about the `.` yet.

---

# 7. What Can a Dockerfile Define?

A Dockerfile can contain instructions for things such as:

```text
Base environment
Application files
Dependencies
Environment variables
Working directory
Startup command
```

For example:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY app.py .

CMD ["python", "app.py"]
```

Here we can already see several Dockerfile instructions:

```text
FROM
WORKDIR
COPY
RUN
CMD
```

We will study them properly when we get to Dockerfile construction.

For now, the important concept is:

> **A Dockerfile describes the steps/configuration Docker uses to produce an image.**

---

# 8. Dockerfile and CI/CD

This is where Dockerfile becomes important for your actual learning goal.

Suppose your repository contains:

```text
my-app/
├── src/
├── tests/
├── requirements.txt
└── Dockerfile
```

A developer pushes code:

```text
Git push
   │
   ▼
Git Repository
   │
   ▼
CI Pipeline
```

The CI pipeline can then execute:

```bash
docker build -t my-app:1.0 .
```

Result:

```text
Git Repository
      │
      ▼
CI Pipeline
      │
      │ docker build
      ▼
Docker Image
```

That image can later be pushed to a registry and deployed.

We aren't studying the registry or deployment yet. Those belong to later phases.

The key point is:

> **The Dockerfile makes the image build reproducible and automatable.**

The same Dockerfile can be used by:

```text
Developer machine
CI server
Build server
```

to define the image-building process.

---

# 9. Important: Dockerfile Does Not Create the Container Directly

A common misunderstanding is:

> Dockerfile → Container

The more accurate flow is:

```text
Dockerfile
     │
     │ BUILD
     ▼
Docker Image
     │
     │ RUN
     ▼
Docker Container
```

Therefore:

```bash
docker build
```

is primarily about **creating an image**.

While:

```bash
docker run
```

is about **creating/starting a container from an image**.

Keep those two operations separate in your mind.

---

# 10. Hands-on Exercise

Let's verify the entire concept.

### Step 1 — Create the directory

```bash
mkdir docker-demo
cd docker-demo
```

### Step 2 — Create Dockerfile

```bash
vi Dockerfile
```

Add:

```dockerfile
FROM nginx
```

### Step 3 — Build the image

```bash
docker build -t my-nginx:1.0 .
```

### Step 4 — Verify the image

```bash
docker images
```

You should find:

```text
my-nginx   1.0
```

### Step 5 — Create a container

```bash
docker run -d --name my-nginx-container my-nginx:1.0
```

### Step 6 — Verify the container

```bash
docker ps
```

You should find:

```text
my-nginx-container
```

### Step 7 — Clean up

```bash
docker rm -f my-nginx-container
docker rmi my-nginx:1.0
```

---

# 11. One Important Observation

Notice that we started with only:

```dockerfile
FROM nginx
```

Yet Docker produced an image.

Why?

Because `FROM nginx` tells Docker to start the image-building process from an existing image.

This introduces an important concept:

```text
Existing Image
      │
      │ Dockerfile
      ▼
New Image
```

We'll eventually study **base images** in Part 7.

For now, just recognize that Dockerfiles can build on top of existing images.

---

# 12. What You Should Understand Before Moving On

You should now be comfortable with these statements:

### 1.

> A Dockerfile is a text file containing instructions for building an image.

### 2.

> `docker build` uses the Dockerfile to build an image.

### 3.

> The image is the build artifact.

### 4.

> `docker run` creates/starts a container from an image.

### 5.

> A Dockerfile is normally stored alongside the application source code in Git.

### 6.

> The same Dockerfile can be used by developers and CI systems to build the image consistently.

The fundamental flow is:

```text
Source Code
     +
Dockerfile
     │
     │ docker build
     ▼
Docker Image
     │
     │ docker run
     ▼
Docker Container
```

---

# Phase 1 Progress

```text
Phase 1 — Docker Fundamentals

01. Container vs Image       ✅
02. Dockerfile               ✅
03. Build Context             ← NEXT
04. Docker Image Layers
05. Docker Image Registry
06. Container Filesystem
07. Base Images
08. Image Tags
```

**Next: Part 3 — Build Context.**

And this time we'll specifically answer the question:

> **What exactly does the `.` mean in `docker build -t my-app:1.0 .`, and what files does Docker actually send/use during a build?**
