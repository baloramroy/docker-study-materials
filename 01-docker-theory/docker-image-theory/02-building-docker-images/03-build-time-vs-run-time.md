# Phase 2 — Build Your First Docker Image

## Part 3 — Build-Time vs Runtime

### Where We Are

In Part 2, we built the first meaningful portion of our Dockerfile:

```dockerfile id="w5q6z8"
FROM python:3.12

WORKDIR /app

COPY app.py .
```

We learned:

```text id="y7x3q4"
FROM
  ↓
Choose the starting image

WORKDIR
  ↓
Choose the working directory

COPY
  ↓
Put application files into the image
```

**Phase 1 callback:** the Dockerfile instructions contribute to the construction of the image. We now need to distinguish between instructions that affect the image **during build** and instructions that define what happens **when a container runs**.

That distinction is the focus of this part.

---

# 1. Two Different Moments in Docker

There are two major moments we need to keep separate:

```text id="3c6y8r"
1. Building the image

2. Running a container
```

They are not the same operation.

When we build:

```bash id="6m0x1f"
docker build -t my-app:1.0 .
```

Docker is **constructing an image**.

When we run:

```bash id="1m2j8s"
docker run my-app:1.0
```

Docker is **creating and starting a container from that image**.

So:

```text id="z9k3tw"
             docker build
                  │
                  ▼
              IMAGE
                  │
                  │ docker run
                  ▼
              CONTAINER
```

This distinction becomes extremely important when we introduce `RUN` and `CMD`.

---

# 2. `RUN` — Execute Something During the Build

The first new instruction is:

```dockerfile id="0v1w9k"
RUN
```

`RUN` tells Docker:

> **Execute a command while building the image.**

For example:

```dockerfile id="q7x3rm"
RUN pip install flask
```

When Docker processes this instruction during:

```bash id="q7f3xy"
docker build ...
```

it executes:

```bash id="w8v6s2"
pip install flask
```

inside the image-building environment.

The important word is:

> **build**

---

# 3. Why Would We Use `RUN`?

Suppose our application requires Flask.

Our application might eventually contain:

```python id="f6j7k2"
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello from Docker!"
```

This application needs Flask.

Our base image gives us Python, but Flask isn't necessarily installed.

So during the image build we could write:

```dockerfile id="0a4r6m"
FROM python:3.12

WORKDIR /app

RUN pip install flask

COPY app.py .
```

Docker processes:

```text id="q4o1i8"
FROM
 ↓
Python environment

WORKDIR
 ↓
/app

RUN pip install flask
 ↓
Flask installed

COPY
 ↓
Application added
```

The resulting image contains Flask.

---

# 4. `RUN` Happens Before a Container Exists

This is the most important thing to understand about `RUN`.

Consider:

```dockerfile id="6cz4qt"
RUN pip install flask
```

At this point, we're still building the image.

There isn't a container created by your eventual `docker run` command yet.

Conceptually:

```text id="q0y7xj"
docker build
    │
    ├── FROM
    │
    ├── WORKDIR
    │
    ├── RUN pip install flask
    │
    └── COPY
    │
    ▼
  IMAGE
    │
    │
    │ later...
    ▼
docker run
    │
    ▼
CONTAINER
```

So `RUN` is a **build-time instruction**.

---

# 5. Our Current Application Doesn't Need `RUN`

Our application is still:

```python id="u2d4i1"
print("Hello from Docker!")
```

It doesn't use Flask or any third-party package.

Python itself is already provided by:

```dockerfile id="7x2v4g"
FROM python:3.12
```

Therefore we don't need:

```dockerfile id="j6v8r3"
RUN pip install ...
```

Our Dockerfile can remain:

```dockerfile id="3i1c7x"
FROM python:3.12

WORKDIR /app

COPY app.py .
```

That's useful because it lets us introduce `RUN` without forcing an unnecessary command into our first image.

---

# 6. `RUN` Is Not "Run My Application"

The name `RUN` can be misleading for beginners.

You might initially think:

```dockerfile id="3z7g1h"
RUN python app.py
```

means:

> "When the container starts, run my application."

It doesn't.

It means:

> "Run `python app.py` **while building the image**."

For example:

```dockerfile id="m5v9s4"
FROM python:3.12

WORKDIR /app

COPY app.py .

RUN python app.py
```

During:

```bash id="9n1p7x"
docker build -t my-app:1.0 .
```

Docker could execute:

```text id="0y5z9c"
python app.py
```

during the build.

That's very different from defining what should happen when we later run:

```bash id="0d7j2v"
docker run my-app:1.0
```

We'll use `CMD` for that.

---

# 7. `CMD` — What Should Happen When the Container Starts?

Now we introduce:

```dockerfile id="l2f4x8"
CMD
```

`CMD` defines the **default command** that should run when a container starts from the image.

For our application:

```dockerfile id="c5n0j2"
CMD ["python", "app.py"]
```

This means:

> When a container starts from this image, the default command is `python app.py`.

Now our Dockerfile becomes:

```dockerfile id="5q3m8v"
FROM python:3.12

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

This is our first complete Dockerfile.

---

# 8. When Does `CMD` Happen?

Unlike `RUN`, `CMD` is associated with **container runtime**.

The flow is:

```text id="7c2x8m"
docker build
     │
     ▼
   IMAGE
     │
     │
     │ docker run
     ▼
 CONTAINER
     │
     ▼
CMD executes
     │
     ▼
python app.py
```

So:

```text id="4j6k1a"
RUN
 ↓
Build time

CMD
 ↓
Container runtime
```

This is the key distinction of Part 3.

---

# 9. `RUN` vs `CMD`

Let's put them directly beside each other.

| Instruction | When?           | Purpose                                       |
| ----------- | --------------- | --------------------------------------------- |
| `RUN`       | Image build     | Execute a command to help construct the image |
| `CMD`       | Container start | Define the default command for the container  |

For example:

```dockerfile id="1p4x8m"
RUN pip install flask
```

means:

```text id="3y9n1h"
Build image
     ↓
Install Flask
     ↓
Save resulting image state
```

Whereas:

```dockerfile id="x7k3p0"
CMD ["python", "app.py"]
```

means:

```text id="1a9m6q"
Start container
     ↓
Run python app.py
```

---

# 10. The Most Important Diagram of Part 3

This is the mental model I want you to remember:

```text id="7y3x8m"
                    BUILD TIME
               docker build ...
                       │
                       ▼
              ┌─────────────────┐
              │     Dockerfile  │
              │                 │
              │ FROM            │
              │ WORKDIR         │
              │ COPY            │
              │ RUN             │
              └────────┬────────┘
                       │
                       ▼
                  DOCKER IMAGE
                  my-app:1.0
                       │
                       │
                  docker run
                       │
                       ▼
                  RUNTIME
                       │
                       ▼
                  CONTAINER
                       │
                       ▼
                     CMD
                       │
                       ▼
                 python app.py
```

The simplest version is:

```text id="5r6k2d"
RUN → BUILD
CMD → RUN
```

Not:

```text
RUN → run the application
```

---

# 11. Why Does `RUN` Affect the Image?

Suppose we write:

```dockerfile id="7n3c1v"
RUN pip install flask
```

After Docker completes the build, Flask is part of the resulting image environment.

So later:

```bash id="a8q4x7"
docker run my-app:1.0
```

doesn't need to execute:

```bash id="h3r8m1"
pip install flask
```

again.

The dependency was installed **when the image was built**.

Conceptually:

```text id="m8v4s0"
BUILD
 │
 ├── Install dependencies
 ├── Copy application
 └── Configure image
          │
          ▼
        IMAGE
          │
          │ docker run
          ▼
      CONTAINER
          │
          ▼
       Application
```

That's one of the major benefits of building images.

---

# 12. Why Does `CMD` Not Affect the Image Filesystem in the Same Way?

Consider:

```dockerfile id="j1w8d3"
CMD ["python", "app.py"]
```

This doesn't install Python.

It doesn't copy `app.py`.

It doesn't create your application environment.

Instead, it tells Docker:

> **When a container is started from this image, this is the default command I want to run.**

So it's primarily defining runtime configuration/behavior.

You can think of it as:

```text id="v2m7q5"
RUN
 ↓
Change/build the image

CMD
 ↓
Define what the container should do at startup
```

---

# 13. A Common Beginner Mistake

Suppose someone writes:

```dockerfile id="7k1s9m"
FROM python:3.12

WORKDIR /app

COPY app.py .

RUN python app.py
```

They build:

```bash id="w7m3k2"
docker build -t my-app:1.0 .
```

They see:

```text id="8q2r5v"
Hello from Docker!
```

and think:

> "Great. My image will run the application."

But that's not what happened.

The application ran **during the build**.

The resulting image does not automatically mean:

```text id="f9z4c1"
docker run
    ↓
python app.py
```

To define that behavior, we need:

```dockerfile id="5v1r7n"
CMD ["python", "app.py"]
```

---

# 14. Build Time and Runtime Are Separate

This distinction is worth seeing as two completely separate phases.

## Build phase

```text id="c1w8x3"
Source code
    │
    ▼
Dockerfile
    │
    ▼
docker build
    │
    ├── FROM
    ├── WORKDIR
    ├── COPY
    ├── RUN
    │
    ▼
Docker Image
```

## Runtime phase

```text id="p6r2m8"
Docker Image
    │
    ▼
docker run
    │
    ▼
Container
    │
    └── CMD
          │
          ▼
      Application
```

This separation is fundamental to Docker.

---

# 15. What Happens If We Change `app.py`?

Suppose our application initially says:

```python id="2z9k4m"
print("Hello from Docker!")
```

We build:

```bash id="l4x8n2"
docker build -t my-app:1.0 .
```

The image contains the copied version of:

```text id="1w3m8a"
/app/app.py
```

Now we change the host file:

```python id="m8y4p1"
print("Hello from my new application!")
```

The already-built image does **not** automatically change.

The image contains its own copy.

We need to build again:

```bash id="j3k7p5"
docker build -t my-app:1.0 .
```

This gives us a new image build containing the updated application.

This reinforces the distinction:

```text id="9w2v6c"
Host source
    │
    │ COPY during build
    ▼
Image
    │
    │ docker run
    ▼
Container
```

---

# 16. Why This Matters for CI/CD

Here's the short CI/CD callback rather than rebuilding the full pipeline:

**CI/CD callback:** later, a CI system will typically take the committed source and Dockerfile and execute essentially the same `docker build` process automatically.

So the distinction remains:

```text id="v8y1k6"
Source + Dockerfile
       │
       │ CI build
       ▼
    Image
       │
       ▼
Deployment
       │
       ▼
Container
```

The important thing for now is that the **image is produced at build time**, while the **application process starts at runtime**.

---

# 17. Our Complete Dockerfile

We can now understand the entire Dockerfile:

```dockerfile id="0f8g2p"
FROM python:3.12

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

Read it from top to bottom:

```text id="5n3v7x"
FROM python:3.12
    ↓
Choose the starting environment.

WORKDIR /app
    ↓
Set /app as the working directory.

COPY app.py .
    ↓
Put our application into /app.

CMD ["python", "app.py"]
    ↓
When the container starts,
run python app.py.
```

Notice the transition:

```text id="5s1j8q"
FROM
WORKDIR
COPY
     ↓
BUILDING THE IMAGE
     ↓
IMAGE
     ↓
CMD
     ↓
RUNNING THE CONTAINER
```

---

# 18. One Important Detail About `CMD`

`CMD` defines a **default** command.

That means the command isn't necessarily impossible to replace when you run the container.

For example, with:

```dockerfile id="k7m4q1"
CMD ["python", "app.py"]
```

the image has a default startup command.

Later, Docker's runtime command can be supplied differently.

We won't go deeply into command overriding yet, because that becomes much easier to understand after we learn `ENTRYPOINT` in Part 4.

For now:

> Think of `CMD` as the image's default runtime command.

---

# 19. Part 3 Checkpoint

You should now be able to answer these questions without looking back.

### When does `RUN` execute?

**During image build.**

```text id="m2n8x4"
docker build
    ↓
RUN
```

### When does `CMD` matter?

**When a container starts.**

```text id="b7q3k1"
docker run
    ↓
container starts
    ↓
CMD
```

### Does `RUN python app.py` define the container's startup command?

**No.**

It executes Python during the image build.

### What defines the default startup command?

```dockerfile id="9x2p6v"
CMD ["python", "app.py"]
```

### The key distinction?

```text id="z3v8m1"
RUN  → BUILD TIME
CMD  → RUNTIME
```

---

# The Mental Model

This is the one diagram to carry forward:

```text id="f2k8q6"
                    BUILD
                      │
                 docker build
                      │
                      ▼
              ┌───────────────┐
              │   Dockerfile  │
              │               │
              │ FROM          │
              │ WORKDIR       │
              │ COPY          │
              │ RUN           │
              └───────┬───────┘
                      │
                      ▼
                DOCKER IMAGE
                  my-app:1.0
                      │
                 docker run
                      │
                      ▼
                  CONTAINER
                      │
                      ▼
                     CMD
                      │
                      ▼
                python app.py
```

The core idea is simple:

> **`RUN` helps build the image. `CMD` tells the container what to run.**

**Part 3 is complete.**

In **Part 4 — Configuration & Container Behavior**, we'll finish the remaining Dockerfile instructions:

```text
ENV
EXPOSE
ENTRYPOINT
```

and then make the important comparison:

```text
CMD vs ENTRYPOINT
```

That will complete our understanding of the main Dockerfile instructions before we actually build the image in Part 5.
