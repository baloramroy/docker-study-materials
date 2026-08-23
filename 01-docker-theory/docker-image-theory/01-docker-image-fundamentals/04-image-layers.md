# Phase 1 — Part 4: Docker Image Layers

## Where we are

```text
Phase 1 — Docker Fundamentals

01. Container vs Image       ✅
02. Dockerfile               ✅
03. Build Context            ✅
04. Docker Image Layers      ← NOW
05. Docker Image Registry
06. Container Filesystem
07. Base Images
08. Image Tags
```

This is one of the **most important Docker fundamentals** for CI/CD because image layers directly affect:

* image size
* build speed
* build caching
* network transfer
* rebuild behavior

We'll understand those progressively.

---

# 1. What is an Image Layer?

A Docker image is **not stored as one single block of data**.

It is built from multiple **layers**.

For example, conceptually:

```text
Docker Image
┌─────────────────────────────┐
│ Application files           │  ← Layer
├─────────────────────────────┤
│ Dependencies                │  ← Layer
├─────────────────────────────┤
│ Base environment            │  ← Layer
└─────────────────────────────┘
```

Each layer represents filesystem changes made during the image build.

A useful mental model is:

> **An image is a stack of filesystem layers.**

---

# 2. Where Do Layers Come From?

Let's use a simple Dockerfile:

```dockerfile
FROM ubuntu:24.04

RUN apt-get update

RUN apt-get install -y nginx

COPY index.html /var/www/html/
```

Conceptually, Docker produces something like:

```text
┌──────────────────────────────┐
│ COPY index.html              │  Layer
├──────────────────────────────┤
│ RUN apt-get install nginx    │  Layer
├──────────────────────────────┤
│ RUN apt-get update           │  Layer
├──────────────────────────────┤
│ ubuntu:24.04                 │  Base image layers
└──────────────────────────────┘
```

The exact internal representation is more nuanced, but this is the correct mental model for learning Docker image construction.

---

# 3. Why Are Layers Useful?

The biggest reason is **reuse**.

Imagine you build two applications:

```text
Application A
    ↓
Python environment
    ↓
Application B
    ↓
Python environment
```

If both images use the same base image, Docker can reuse the existing base-image layers instead of storing another completely independent copy.

Conceptually:

```text
             Python Base Layers
              /            \
             /              \
            ▼                ▼
       App A layers      App B layers
```

This saves storage and can reduce the amount of data that needs to be transferred.

---

# 4. Dockerfile Instructions and Layers

A common beginner explanation is:

> "Every Dockerfile instruction creates a layer."

That is **too simplistic**.

For learning purposes, you'll often see instructions such as:

```dockerfile
RUN ...
COPY ...
ADD ...
```

described as creating filesystem layers.

However, Dockerfile instructions also include configuration instructions such as:

```dockerfile
ENV
WORKDIR
EXPOSE
CMD
ENTRYPOINT
```

which don't all behave like ordinary filesystem-change layers.

So the better mental model is:

> **Instructions that modify the image filesystem can produce filesystem layers, while other instructions may contribute image configuration.**

This distinction will matter later when we inspect images.

For now, don't memorize a one-to-one rule between every Dockerfile instruction and a layer.

---

# 5. A Practical Example

Create:

```text
docker-layers/
├── Dockerfile
└── app.txt
```

Dockerfile:

```dockerfile
FROM alpine

RUN echo "Installing something"

COPY app.txt /app/
```

Build:

```bash
docker build -t layers-demo:1.0 .
```

Conceptually:

```text
FROM alpine
     │
     ▼
Alpine base image
     │
     ▼
RUN echo ...
     │
     ▼
New filesystem change
     │
     ▼
COPY app.txt
     │
     ▼
Another filesystem change
     │
     ▼
Final image
```

You can inspect the image history:

```bash
docker history layers-demo:1.0
```

You'll see entries corresponding to the image's build history.

For example, conceptually:

```text
IMAGE        CREATED        CREATED BY
xxxxxxxx     ...            COPY app.txt /app/
xxxxxxxx     ...            RUN echo ...
xxxxxxxx     ...            /bin/sh
```

The exact output depends on the Docker version and base image.

---

# 6. Layers Are Reusable

Suppose you build:

```text
app1
```

from:

```text
alpine
```

and later build:

```text
app2
```

also from:

```text
alpine
```

Docker can reuse the relevant existing layers.

Conceptually:

```text
                 Alpine
                   │
             ┌─────┴─────┐
             ▼           ▼
          App 1         App 2
```

This is one reason container images can be efficient despite being composed of multiple layers.

---

# 7. Layers and Build Cache

This is where layers become particularly important for CI/CD.

Consider:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY app.py .

CMD ["python", "app.py"]
```

Suppose you build it once.

Docker can cache the results of previous build steps.

Now you change only:

```text
app.py
```

and build again.

The earlier parts may still be reusable:

```text
FROM python:3.12          ← reuse
WORKDIR /app              ← reuse
COPY requirements.txt     ← reuse
RUN pip install ...       ← reuse
COPY app.py               ← rebuild
```

Conceptually:

```text
FROM python       ✅ cached
      ↓
WORKDIR            ✅ cached
      ↓
COPY requirements  ✅ cached
      ↓
RUN pip install     ✅ cached
      ↓
COPY app.py         🔨 rebuild
```

This can make subsequent builds dramatically faster.

---

# 8. Why Dockerfile Order Matters

Now consider this Dockerfile:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY . .

RUN pip install -r requirements.txt
```

If you change:

```text
app.py
```

the:

```dockerfile
COPY . .
```

step changes.

That can invalidate the cache for subsequent steps, including:

```dockerfile
RUN pip install -r requirements.txt
```

Even though your dependencies didn't change.

Compare that with:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .
```

Now a change to `app.py` can allow the dependency-installation step to remain cached.

This is why Dockerfile ordering matters.

We'll later study this more deeply when we cover image optimization.

---

# 9. Layer Cache Is Not the Same as Image Layers

Be careful with terminology.

These concepts are related but not identical:

```text
Image Layers
      +
Build Cache
```

**Image layers** are part of the image's storage representation.

**Build cache** is Docker's mechanism for reusing previous build results.

The existence of layers makes efficient reuse possible, but caching involves additional build-system behavior.

For now:

```text
Image layers → structure of image
Build cache  → reuse of previous build work
```

We'll revisit caching later.

---

# 10. Layers Are Immutable

A useful property of Docker image layers is that existing image layers are treated as **immutable**.

Suppose you have:

```text
Layer 1
Layer 2
Layer 3
```

and then make another change.

Docker doesn't normally modify Layer 2 directly.

Instead, a new layer can represent the new filesystem changes:

```text
Layer 1
Layer 2
Layer 3
Layer 4  ← new changes
```

This contributes to Docker's efficient reuse model.

---

# 11. What Happens When You Delete Something?

This is a subtle but important concept.

Suppose:

```dockerfile
RUN echo "secret data" > /tmp/secret
RUN rm /tmp/secret
```

You might think:

> "The secret was created and then deleted, so it can't exist in the image."

But the earlier layer may still contain the file's data.

Conceptually:

```text
Layer 1
└── secret created

Layer 2
└── secret removed
```

The final filesystem view may not show the file, but the data may still exist in an earlier layer.

This is one reason you should **not put secrets into Docker image build steps and then try to delete them afterward**.

We'll revisit secure build secrets later.

---

# 12. Image Layers vs Container Changes

There's another important distinction.

When you run an image:

```text
Docker Image
├── Layer 1
├── Layer 2
└── Layer 3
```

and create a container, the container gets a **writable layer on top** of the image's read-only layers.

Conceptually:

```text
Container
┌──────────────────────────────┐
│ Writable container layer     │
├──────────────────────────────┤
│ Image layer                  │
├──────────────────────────────┤
│ Image layer                  │
├──────────────────────────────┤
│ Image layer                  │
└──────────────────────────────┘
```

We are going to study the container filesystem properly in **Part 6**, so don't go too deeply into this yet.

For now, remember:

> **Image layers belong to the image; the running container has its own writable layer above them.**

---

# 13. Hands-on Exercise

Let's inspect an actual image.

Pull Alpine:

```bash
docker pull alpine
```

Inspect its history:

```bash
docker history alpine
```

Now create:

```text
docker-layers/
└── Dockerfile
```

Dockerfile:

```dockerfile
FROM alpine

RUN echo "First layer"

RUN echo "Second layer"

RUN echo "Third layer"
```

Build:

```bash
docker build -t layers-demo:1.0 .
```

Now:

```bash
docker history layers-demo:1.0
```

You should see your build commands represented in the image history.

The exact output will vary, but conceptually you're seeing:

```text
Final image
     │
     ├── RUN echo "Third layer"
     ├── RUN echo "Second layer"
     ├── RUN echo "First layer"
     └── Alpine base image
```

This is your first direct look at how a Dockerfile contributes to image construction.

---

# 14. Hands-on Exercise — Observe Cache

Now build again:

```bash
docker build -t layers-demo:1.0 .
```

Docker should be able to reuse previously completed work.

You'll see output indicating cached steps, depending on your Docker/BuildKit version.

Now modify the Dockerfile:

```dockerfile
FROM alpine

RUN echo "First layer"

RUN echo "CHANGED"

RUN echo "Third layer"
```

Build again:

```bash
docker build -t layers-demo:1.0 .
```

Observe which steps are rebuilt and which can still be reused.

This demonstrates why Docker's build process isn't simply:

```text
Run everything from the beginning
```

Instead, Docker can reuse previously completed build work when the relevant inputs haven't changed.

---

# 15. The CI/CD Connection

This is where image layers become directly relevant to your learning goal.

Imagine a CI server builds your application every time developers push code:

```text
Developer
    │
    │ git push
    ▼
CI Pipeline
    │
    │ docker build
    ▼
Docker Image
```

If every build had to completely redo expensive operations, builds could become slow.

With effective layer/cache usage:

```text
Build #1
──────────────
All steps → build


Build #2
──────────────
Most steps → cached
One step   → rebuild


Build #3
──────────────
Most steps → cached
One step   → rebuild
```

This is why understanding layers is important when designing Dockerfiles for CI/CD.

---

# 16. Common Mistakes

### Mistake 1 — Thinking the image is one single file

Conceptually:

```text
❌ Image = one giant block
```

Better:

```text
✅ Image = multiple layers + image configuration
```

---

### Mistake 2 — Assuming every Dockerfile instruction is a filesystem layer

Don't memorize:

```text
Every instruction = one layer
```

That's an oversimplification.

Instead:

```text
Filesystem-changing instructions
        ↓
Filesystem layers

Configuration instructions
        ↓
Image configuration
```

---

### Mistake 3 — Thinking deleting a file removes it from all layers

It doesn't necessarily.

A later layer can hide/remove a file from the final filesystem view while earlier layer data remains.

---

### Mistake 4 — Ignoring layer/cache behavior in CI

A Dockerfile that causes expensive steps to rebuild unnecessarily can make CI pipelines much slower.

---

# 17. The Mental Model

Our model is now:

```text
Dockerfile
    │
    │ build instructions
    ▼
┌───────────────────────────┐
│       Docker Image        │
│                           │
│   Layer 3  ← application  │
│   Layer 2  ← dependencies │
│   Layer 1  ← base         │
└───────────────────────────┘
             │
             │ docker run
             ▼
        Docker Container
```

And when rebuilding:

```text
Previous Build
      │
      ▼
Build Cache
      │
      ├── reusable steps → cached
      │
      └── changed steps  → rebuild
```

---

# 18. What You Should Know Now

You should now understand:

* A Docker image consists of multiple layers.
* Layers represent filesystem changes from the image build.
* Base images themselves are composed of layers.
* Layers can be reused between images.
* Docker can reuse previous build results through caching.
* Changing an earlier build step can cause later steps to rebuild.
* Dockerfile instruction ordering therefore matters.
* Image layers are immutable.
* A container adds a writable layer above the image layers.
* Removing a file in a later layer doesn't necessarily erase its data from earlier layers.

The most important relationship is:

```text
Dockerfile
    │
    ▼
Build Steps
    │
    ▼
Image Layers
    │
    ▼
Docker Image
```

---

# Phase 1 Progress

```text
Phase 1 — Docker Fundamentals

01. Container vs Image       ✅
02. Dockerfile               ✅
03. Build Context            ✅
04. Docker Image Layers      ✅
05. Docker Image Registry    ← NEXT
06. Container Filesystem
07. Base Images
08. Image Tags
```

**Next: Part 5 — Docker Image Registry.**

There we'll connect the image we've been building locally to the next major CI/CD concept:

```text
Build Image
    ↓
Store Image
    ↓
Share Image
    ↓
Pull Image
    ↓
Deploy
```

And we'll distinguish **Docker Hub, private registries, image repositories, tags, and image digests**—while keeping the deeper tag discussion for Part 8 as planned.
