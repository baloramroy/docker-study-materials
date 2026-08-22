Absolutely. We’ll continue **strictly in roadmap order** and keep the same revised-explanation style we established in Phase 1.

# Phase 2 — Build Your First Docker Image

## Where we are

Phase 1 established the concepts:

```text
Phase 1 — Fundamentals

01. Container vs Image       ✅
02. Dockerfile               ✅
03. Build Context             ✅
04. Docker Image Layers       ✅
05. Docker Image Registry
06. Container Filesystem
07. Base Images
08. Image Tags
```

Now we move to:

```text
Phase 2 — Build Your First Image
```

The purpose of this phase is **not yet to optimize images or build CI pipelines**.

Our goal is simple:

> **Take application source code → write a Dockerfile → build an image → inspect the image → run a container from it.**

Once you can do this confidently, the later CI/CD process will make much more sense.

---

# 1. What are we actually going to build?

We'll use a very small Python application.

Our project will look like this:

```text
my-app/
├── app.py
├── requirements.txt
└── Dockerfile
```

There are three important files.

### `app.py`

This is our actual application.

### `requirements.txt`

This tells Python which external packages the application needs.

### `Dockerfile`

This tells Docker:

> "How should I construct an image that can run this application?"

So conceptually:

```text
Application Source
       │
       ├── app.py
       ├── requirements.txt
       │
       ▼
   Dockerfile
       │
       │ docker build
       ▼
   Docker Image
       │
       │ docker run
       ▼
   Container
```

That is the entire objective of this phase.

---

# 2. First understand the responsibility of each component

This is important.

Suppose we have:

```text
app.py
```

Docker doesn't automatically know what this file is.

Docker doesn't automatically know:

* which Python version we need
* where the application should live
* which dependencies must be installed
* which command should start the application

We need to provide those instructions.

That's the job of the **Dockerfile**.

Think of it this way:

```text
app.py
    │
    │ "This is my application"
    ▼
Dockerfile
    │
    │ "Here is how to package it"
    ▼
Docker Image
    │
    │ "Here is the packaged application environment"
    ▼
Container
    │
    │ "Now run it"
    ▼
Running Application
```

---

# 3. Our first application

We'll keep the application extremely simple.

`app.py` could contain:

```python
print("Hello from Docker!")
```

There is nothing special here.

If you run it directly on a machine:

```bash
python app.py
```

you get:

```text
Hello from Docker!
```

Now our goal is to make Docker run the same application.

---

# 4. Why do we need a Dockerfile?

Imagine you give Docker only:

```text
app.py
```

Docker doesn't know:

```text
Which operating system?
Which Python?
Where should app.py go?
Does Python need dependencies?
What command starts the application?
```

So we create:

```text
Dockerfile
```

The Dockerfile becomes the **recipe for creating our image**.

For example:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

Don't worry if every instruction isn't completely clear yet.

We'll understand each one carefully.

---

# 5. Understanding the Dockerfile line by line

## `FROM`

```dockerfile
FROM python:3.12
```

This says:

> Start building my image from the `python:3.12` image.

Remember our Phase 1 discussion about **base images**.

We're saying:

```text
Existing Python Image
        │
        ▼
   Our Dockerfile
        │
        ▼
   Our final image
```

We aren't building an operating system from zero.

We're starting from an existing image.

---

# 6. `WORKDIR`

Next:

```dockerfile
WORKDIR /app
```

This tells Docker:

> Make `/app` the working directory for subsequent instructions.

So instead of putting our application somewhere random, we establish:

```text
/app
```

as the application's working directory.

Conceptually:

```text
Container filesystem

/
├── ...
├── app/
│   └──
└── ...
```

After:

```dockerfile
WORKDIR /app
```

Docker treats `/app` as the current working directory.

---

# 7. `COPY`

Now:

```dockerfile
COPY app.py .
```

This means:

> Copy `app.py` from the build context into the current working directory inside the image.

Because our current working directory is:

```text
/app
```

the result is:

```text
/app/app.py
```

So:

```text
Host/build context

my-app/
└── app.py
      │
      │ COPY
      ▼
Image

/app/
└── app.py
```

This is one of the most important ideas in Docker image building.

We're taking **application source from the build context** and putting it into the image.

---

# 8. `CMD`

Finally:

```dockerfile
CMD ["python", "app.py"]
```

This tells Docker:

> When a container is started from this image, the default command should be `python app.py`.

Notice something important:

`CMD` does **not** execute while we're building the image.

It defines the default command for the **container**.

This distinction is extremely important.

### During build

```text
docker build
     │
     ├── FROM
     ├── WORKDIR
     └── COPY
```

### During container startup

```text
docker run
     │
     ▼
CMD ["python", "app.py"]
     │
     ▼
python app.py
```

So:

> `Dockerfile` describes how to construct the image, while `CMD` describes the default process to run when a container starts.

---

# 9. Our first complete Dockerfile

Putting those instructions together:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

At this point, don't try to memorize it.

Understand the flow:

```text
FROM
 ↓
Get Python environment

WORKDIR
 ↓
Choose application directory

COPY
 ↓
Put application into image

CMD
 ↓
Define default startup command
```

---

# 10. Now we build the image

From inside:

```text
my-app/
```

we run:

```bash
docker build -t my-app:1.0 .
```

Let's break this command apart.

## `docker`

The Docker CLI.

## `build`

We're asking Docker to build an image.

## `-t`

Means:

> Give the resulting image a name/tag.

## `my-app:1.0`

Our image name and tag.

Conceptually:

```text
repository = my-app
tag        = 1.0
```

## `.`

This is extremely important.

The `.` means:

> Use the current directory as the build context.

So:

```bash
docker build -t my-app:1.0 .
                              ↑
                         build context
```

This connects directly to what we learned in Phase 1.

---

# 11. What happens internally during `docker build`?

This is the most important mental model of Phase 2.

When you run:

```bash
docker build -t my-app:1.0 .
```

Docker essentially goes through this process:

```text
Current directory
       │
       ▼
Build context
       │
       ▼
Read Dockerfile
       │
       ▼
FROM python:3.12
       │
       ▼
WORKDIR /app
       │
       ▼
COPY app.py .
       │
       ▼
CMD ["python", "app.py"]
       │
       ▼
Final Docker Image
```

So `docker build` is **not running your application**.

It's constructing an image according to the Dockerfile.

---

# 12. Verify that the image exists

After the build:

```bash
docker images
```

You should see something conceptually like:

```text
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
my-app        1.0       abc123...      ...            ...
```

Now Docker has an image called:

```text
my-app:1.0
```

This is our packaged application environment.

---

# 13. Image vs Container — again

This is where the Phase 1 concept becomes practical.

We currently have:

```text
my-app:1.0
```

That's an **image**.

It isn't running.

To create a container from it:

```bash
docker run my-app:1.0
```

Now:

```text
Image
  │
  │ docker run
  ▼
Container
  │
  ▼
python app.py
```

The container starts and executes:

```text
python app.py
```

because that's what we specified with:

```dockerfile
CMD ["python", "app.py"]
```

The output should be:

```text
Hello from Docker!
```

---

# 14. The complete workflow

At this point, you should understand this entire chain:

```text
                 SOURCE CODE
                     │
                     ▼
              ┌──────────────┐
              │  Dockerfile  │
              └──────┬───────┘
                     │
                     │ docker build
                     ▼
              ┌──────────────┐
              │ Docker Image │
              │  my-app:1.0  │
              └──────┬───────┘
                     │
                     │ docker run
                     ▼
              ┌──────────────┐
              │   Container  │
              └──────┬───────┘
                     │
                     ▼
                python app.py
```

This is the fundamental Docker image-building workflow.

---

# 15. But our example is intentionally incomplete

There is an important thing we haven't used yet:

```text
requirements.txt
```

Suppose our application eventually becomes:

```python
from flask import Flask
```

Now Python needs Flask installed.

Our project becomes:

```text
my-app/
├── app.py
├── requirements.txt
└── Dockerfile
```

`requirements.txt`:

```text
Flask
```

Now our Dockerfile needs another step:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY app.py .

CMD ["python", "app.py"]
```

Notice the new instruction:

```dockerfile
RUN
```

This is an important distinction.

---

# 16. `RUN` vs `CMD`

These two are commonly confused.

### `RUN`

```dockerfile
RUN pip install -r requirements.txt
```

`RUN` executes **during image building**.

So:

```text
docker build
     │
     ▼
RUN pip install
     │
     ▼
Package gets installed into image
```

### `CMD`

```dockerfile
CMD ["python", "app.py"]
```

`CMD` defines the default command when the **container starts**.

So:

```text
docker run
     │
     ▼
CMD
     │
     ▼
python app.py
```

Remember this distinction:

> **RUN = build time**

> **CMD = container runtime**

This distinction will become extremely important later when we discuss CI/CD.

---

# 17. The basic Dockerfile instructions we'll learn

Our Phase 2 roadmap introduced:

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

We have now introduced:

```text
FROM       → base image
WORKDIR    → working directory
COPY       → copy files into image
RUN        → execute something during build
CMD        → default container command
```

We have **not yet gone deeply into**:

```text
ENV
EXPOSE
ENTRYPOINT
```

And that's intentional.

We should learn them one by one rather than throw everything together.

---

# 18. Why we're learning this manually before CI/CD

Eventually your Jenkins pipeline may contain something like:

```text
Checkout source
     ↓
docker build
     ↓
docker tag
     ↓
docker push
```

But if you don't understand what `docker build` actually does, then a Jenkins pipeline can become just a collection of commands you copied.

Our objective is different.

We want you to eventually look at:

```bash
docker build -t my-app:1.0 .
```

and understand:

```text
docker build
     │
     ├── Build context
     │
     ├── Dockerfile
     │
     ├── Base image
     │
     ├── Instructions
     │
     ├── Layers
     │
     ├── Cache
     │
     └── Final image
```

Then CI/CD simply automates this process.

---

# 19. Phase 2 mental model

For now, remember this:

```text
              Dockerfile
                  │
                  │ instructions
                  ▼
Source ──────► Docker Build
                  │
                  ▼
             Docker Image
                  │
                  │ docker run
                  ▼
              Container
                  │
                  ▼
             Application
```

And the five instructions we've covered so far:

| Instruction | Main purpose               | When    |
| ----------- | -------------------------- | ------- |
| `FROM`      | Select base image          | Build   |
| `WORKDIR`   | Set working directory      | Build   |
| `COPY`      | Copy files into image      | Build   |
| `RUN`       | Execute build-time command | Build   |
| `CMD`       | Default startup command    | Runtime |

---

## Phase 2 progress

```text
Phase 2 — Build Your First Image

01. Create simple application       ← DONE
02. Understand Dockerfile            ← DONE
03. FROM                             ← DONE
04. WORKDIR                          ← DONE
05. COPY                             ← DONE
06. RUN                              ← INTRODUCED
07. CMD                              ← DONE
08. ENV                              ← NEXT
09. EXPOSE                           ← NEXT
10. ENTRYPOINT                       ← NEXT
11. docker build                     ← DONE
12. Inspect image                    ← DONE
13. Run container                    ← DONE
```

**Next:** we'll continue Phase 2 with **`ENV`, `EXPOSE`, and `ENTRYPOINT`**, while carefully comparing them with `RUN` and `CMD`. We won't jump to image optimization, multi-stage builds, registries, or CI/CD until the Phase 2 roadmap is complete.
