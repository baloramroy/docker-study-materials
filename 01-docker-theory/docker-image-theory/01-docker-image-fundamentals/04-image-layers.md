# Phase 1 — Part 4: Docker Image Layers

We are now moving to:

```text
01. Container vs Image       ✅
02. Dockerfile               ✅
03. Build context            ✅
04. Docker image layers      ← NOW
05. Docker image registry
06. Container filesystem
07. Base images
08. Image tags
```

This topic is **very important for CI/CD**, because Docker image layers are directly connected to:

* build caching
* build speed
* image size
* Dockerfile ordering
* CI pipeline performance

---

# 1. What Is a Docker Image Layer?

A Docker image is not just one giant block of files.

It is built from **multiple layers**.

Think of it like this:

```text
Docker Image
│
├── Layer 1
├── Layer 2
├── Layer 3
├── Layer 4
└── Image configuration
```

A simplified example:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY app.py .

CMD ["python", "app.py"]
```

Conceptually, the image can be thought of as:

```text
┌─────────────────────────────┐
│        app.py layer         │
├─────────────────────────────┤
│    pip install layer        │
├─────────────────────────────┤
│    requirements.txt layer   │
├─────────────────────────────┤
│       WORKDIR layer         │
├─────────────────────────────┤
│      Python base image      │
└─────────────────────────────┘
```

Each layer represents filesystem changes produced during the build.

---

# 2. Why Does Docker Use Layers?

The biggest reason is **reuse**.

Imagine you build:

```text
myapp:1.0
```

and later build:

```text
myapp:1.1
```

If most of the image hasn't changed, Docker doesn't need to rebuild everything from scratch.

It can reuse layers that are still valid.

Conceptually:

```text
Build 1

Layer A ──┐
Layer B ──┤
Layer C ──┤
Layer D ──┘
           ↓
        Image 1
```

Then:

```text
Build 2

Layer A ── reused
Layer B ── reused
Layer C ── reused
Layer D ── changed
           ↓
        Image 2
```

This is the foundation of Docker's **build cache**.

We'll study caching more deeply after this topic.

---

# 3. Dockerfile Instructions and Layers

At a beginner level, it is useful to associate filesystem-changing Dockerfile instructions with layers.

For example:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY app.py .
```

Think:

```text
FROM
 ↓
Base image

WORKDIR
 ↓
Filesystem/configuration change

COPY
 ↓
Layer

RUN
 ↓
Layer

COPY
 ↓
Layer
```

But there is an important nuance:

> **Not every Dockerfile instruction necessarily creates a filesystem layer.**

For example:

```dockerfile
CMD ["python", "app.py"]
```

doesn't create a normal filesystem layer in the same way as `RUN` or `COPY`. It contributes to the image's configuration.

So don't memorize:

> "Every Dockerfile instruction = one layer."

That's too simplistic.

A better model is:

```text
Dockerfile instruction
        │
        ▼
Does it change the image filesystem?
        │
        ├── Yes → filesystem layer/change
        │
        └── No  → may affect image configuration
```

---

# 4. Let's Build an Example

Suppose we have:

```text
my-app/
├── Dockerfile
├── app.py
└── requirements.txt
```

Dockerfile:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY app.py .

CMD ["python", "app.py"]
```

Run:

```bash
docker build -t myapp:1.0 .
```

Conceptually:

```text
                Dockerfile
                    │
                    ▼
              Build process
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼
      FROM         COPY          RUN
       │            │             │
       ▼            ▼             ▼
  Base image    requirements   Python deps
       │
       └──────────────┬───────────────┐
                      ▼               ▼
                  COPY app.py      CMD config
                      │
                      ▼
                 Final image
```

---

# 5. The Base Image Is Already Layered

This is another important concept.

When you write:

```dockerfile
FROM python:3.12
```

you're not necessarily starting with an empty filesystem.

The Python image itself is already built from layers.

Conceptually:

```text
Your Image
│
├── Your application layer
├── Your dependency layer
├── Your configuration/files
│
└── python:3.12
      │
      ├── Python layer
      ├── OS filesystem layer
      └── Other base layers
```

So your final image is effectively built **on top of existing image layers**.

This is one reason base-image selection matters.

---

# 6. Image Layers Are Immutable

This is a very important concept.

Once a layer has been created, Docker doesn't normally modify that existing layer.

Instead, a new change produces another layer.

Think:

```text
Layer 1
   ↓
Layer 2
   ↓
Layer 3
```

rather than:

```text
Layer 1
   ↓
modify Layer 1
   ↓
Layer 1 changed
```

Conceptually:

```text
Original

┌─────────────┐
│   Layer 3   │
├─────────────┤
│   Layer 2   │
├─────────────┤
│   Layer 1   │
└─────────────┘
```

If another build changes something:

```text
┌─────────────┐
│   Layer 4   │ ← new
├─────────────┤
│   Layer 3   │ ← reused
├─────────────┤
│   Layer 2   │ ← reused
├─────────────┤
│   Layer 1   │ ← reused
└─────────────┘
```

This is what makes layer reuse possible.

---

# 7. The Big CI/CD Advantage

Imagine this Dockerfile:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY app.py .

CMD ["python", "app.py"]
```

Suppose you change only:

```text
app.py
```

Your dependencies haven't changed.

Docker can potentially reuse:

```text
FROM python:3.12
        ↓
WORKDIR /app
        ↓
COPY requirements.txt
        ↓
RUN pip install
```

and only rebuild the part affected by:

```text
COPY app.py
```

Conceptually:

```text
             Build #1
                │
                ▼
┌─────────────────────────┐
│ Python base             │ ← cached
├─────────────────────────┤
│ requirements.txt        │ ← cached
├─────────────────────────┤
│ pip install             │ ← cached
├─────────────────────────┤
│ app.py                  │ ← changed
└─────────────────────────┘
```

This can make a huge difference in CI/CD.

---

# 8. Why Dockerfile Ordering Matters

Now we're getting to one of the most important practical lessons.

Compare these two Dockerfiles.

### Version A

```dockerfile
FROM python:3.12

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY app.py .

CMD ["python", "app.py"]
```

### Version B

```dockerfile
FROM python:3.12

WORKDIR /app

COPY . .

RUN pip install -r requirements.txt

CMD ["python", "app.py"]
```

Suppose only `app.py` changes.

With Version A:

```text
requirements.txt
      ↓
pip install
      ↓
cached
```

So Docker can reuse the dependency installation.

With Version B:

```text
COPY .
   ↓
app.py changed
   ↓
COPY layer changed
   ↓
RUN pip install
   ↓
cache invalidated
   ↓
dependencies installed again
```

Therefore:

> **Dockerfile instruction ordering can have a major impact on build performance.**

This becomes extremely important in CI pipelines.

---

# 9. Think of Layers as a Stack

A useful mental model is:

```text
             Top
              │
       ┌─────────────┐
       │ Application │
       ├─────────────┤
       │ Dependencies│
       ├─────────────┤
       │ Configuration
       ├─────────────┤
       │ Base image  │
       └─────────────┘
              │
             Base
```

The image is constructed from the bottom upward.

Each layer adds filesystem changes.

---

# 10. What Happens When a File Is Deleted?

This is an interesting consequence of layered images.

Suppose:

```dockerfile
RUN echo "secret" > /tmp/secret.txt
```

creates a file in one layer.

Then later:

```dockerfile
RUN rm /tmp/secret.txt
```

removes it from the **current filesystem view**.

But the original layer still contains the file.

Conceptually:

```text
Layer 2
┌─────────────────────┐
│ secret.txt          │
└─────────────────────┘
          ↓
Layer 3
┌─────────────────────┐
│ deletion of file    │
└─────────────────────┘
```

The final container filesystem may no longer show:

```text
/tmp/secret.txt
```

but the data may still exist in the underlying image history/layers.

This is one reason you should **never put secrets into an image and assume deleting them later makes them safe**.

Bad:

```dockerfile
RUN echo "my-password" > /tmp/password
RUN rm /tmp/password
```

The correct principle is:

> **Don't introduce secrets into image layers in the first place.**

We'll cover proper build secrets later.

---

# 11. Image Layers vs Container Writable Layer

There's another distinction we'll eventually explore in the **Container Filesystem** section.

When you create a container from an image:

```text
Docker Image
    │
    ├── Layer 1
    ├── Layer 2
    ├── Layer 3
    └── Layer 4
           │
           ▼
       Container
           │
           ▼
    Writable layer
```

The image layers are normally read-only.

The running container gets a writable layer on top.

So conceptually:

```text
Container
│
├── Writable container layer
│
├── Image layer 3
├── Image layer 2
└── Image layer 1
```

We'll study this properly later.

For now, remember:

> **Image layers form the immutable image; a running container gets its own writable layer.**

---

# 12. How to See Image Layers

You don't have to rely only on theory.

Docker provides:

```bash
docker image history myapp:1.0
```

For example:

```bash
docker image history myapp:1.0
```

This shows the image's build history.

You'll see information corresponding to commands such as:

```text
CMD
COPY
RUN
WORKDIR
FROM
```

The exact output depends on the image and builder.

This is one of the best commands for understanding how an image was constructed.

---

# 13. `docker image inspect`

Another useful command is:

```bash
docker image inspect myapp:1.0
```

This provides detailed image metadata.

You'll eventually encounter information such as:

```text
Id
RepoTags
RepoDigests
RootFS
Config
Architecture
OS
```

The `RootFS` information is particularly relevant to understanding the image's layers.

Don't worry about every field yet.

We'll use this command more as we progress.

---

# 14. Layer Sharing

One of Docker's biggest advantages is that different images can share layers.

Imagine:

```text
python:3.12
      │
      ├──────────────┐
      │              │
      ▼              ▼
   myapp:1.0      myapp:2.0
```

Both applications might use:

```text
python:3.12
```

So they can potentially share the same underlying base layers.

Conceptually:

```text
              python base layers
                    │
             ┌──────┴──────┐
             ▼             ▼
          myapp:1.0     myapp:2.0
```

Docker doesn't necessarily need to store duplicate copies of identical layers.

This reduces storage requirements.

---

# 15. Layer Reuse in CI/CD

Now connect everything together.

Suppose Jenkins builds:

```text
myapp:1.0
```

Monday:

```text
FROM python:3.12
COPY requirements.txt .
RUN pip install ...
COPY app.py .
```

Tuesday:

```text
app.py changed
```

A well-configured build environment may be able to reuse:

```text
Python base
       ↓
requirements
       ↓
installed dependencies
```

and only rebuild the application layer.

This gives:

```text
Less work
   ↓
Faster Docker build
   ↓
Faster CI pipeline
   ↓
Faster deployment cycle
```

That's why understanding layers isn't just Docker theory.

It's directly relevant to your CI/CD goal.

---

# 16. A Critical Mental Model

Don't think:

```text
Dockerfile
    ↓
one giant image
```

Think:

```text
Dockerfile
    ↓
sequence of filesystem changes
    ↓
multiple image layers
    ↓
final image
```

For example:

```text
FROM
 ↓
Base layers

COPY requirements.txt
 ↓
Layer

RUN pip install
 ↓
Layer

COPY app.py
 ↓
Layer

Configuration
 ↓
Final image
```

---

# 17. But Don't Oversimplify Layers

One correction to the beginner mental model:

It's tempting to say:

> "Every `RUN`, `COPY`, and `ADD` creates exactly one layer."

That's a useful **introductory approximation**, but modern Docker uses BuildKit and more sophisticated build mechanisms.

The more accurate concept is:

> **Docker builds an image from filesystem snapshots/layers and image configuration, with the builder determining how those changes are represented.**

For your current phase, the practical model is enough:

```text
RUN / COPY / ADD
        ↓
filesystem changes
        ↓
image layers
```

Later, when we study **BuildKit**, we'll refine this model.

---

# 18. The Most Important Connection: Layers → Cache

Your next major concept after understanding layers is:

```text
Image Layers
     ↓
Build Cache
     ↓
Dockerfile optimization
```

For example:

```dockerfile
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .
```

is usually structured this way because:

```text
requirements.txt changes rarely
        ↓
dependency layer remains reusable

application source changes frequently
        ↓
application layer rebuilds
```

Whereas:

```dockerfile
COPY . .
RUN pip install -r requirements.txt
```

can cause dependency installation to be repeated when unrelated application files change.

So:

```text
Good Dockerfile ordering
          ↓
Better cache reuse
          ↓
Faster CI builds
```

---

# 19. Your Core Mental Model

Remember this diagram:

```text
                    Dockerfile
                        │
                        ▼
                Build instructions
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
        FROM          COPY            RUN
          │             │             │
          ▼             ▼             ▼
     Base layers    File changes   File changes
          │             │             │
          └─────────────┼─────────────┘
                        ▼
                  Image layers
                        │
                        ▼
                  Docker Image
                        │
                        ▼
                   Container
                        │
                        ▼
                Writable layer
```

The most important sentence for this lesson:

> **A Docker image is composed of immutable filesystem layers built from the image's base and the changes introduced during the build.**

And the CI/CD connection:

> **Because layers can be reused, Docker can avoid repeating unchanged work, making Dockerfile ordering and build caching critical for fast CI/CD builds.**

---

# Phase 1 Progress

```text
01. Container vs Image       ✅
02. Dockerfile               ✅
03. Build context            ✅
04. Docker image layers      ✅
05. Docker image registry    ← NEXT
06. Container filesystem
07. Base images
08. Image tags
```

Next we'll cover **Docker Image Registry**: what a registry actually is, how it differs from Docker Hub, what `docker push`/`docker pull` really do, and how the registry fits into your eventual Jenkins → Registry → Kubernetes CI/CD flow.
