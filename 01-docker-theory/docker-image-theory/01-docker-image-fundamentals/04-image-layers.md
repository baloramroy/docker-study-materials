# Phase 1 — Part 4: Docker Image Layers


First understand this:

> **A Docker image is basically a stack of filesystem changes.**

That is the core idea.

---

## 1. Start with a normal filesystem

Imagine an empty filesystem:

```text
/
├── bin/
├── etc/
├── home/
├── tmp/
└── ...
```

Now imagine Docker says:

> "Start with Ubuntu."

The Ubuntu image already contains a filesystem.

Conceptually:

```text
Ubuntu base image

/
├── bin/
├── etc/
├── home/
├── usr/
├── var/
└── ...
```

This becomes our starting point.

Think of it as:

```text
Layer 1
────────────
Ubuntu filesystem
```

---

# 2. Now we add something

Suppose our Dockerfile says:

```dockerfile
FROM ubuntu:24.04

RUN mkdir /app
```

Docker starts with:

```text
Layer 1
────────────
Ubuntu filesystem
```

Then:

```dockerfile
RUN mkdir /app
```

creates `/app`.

So Docker records a filesystem change:

```text
Layer 2
────────────
Added:
/app/
```

Now the image conceptually looks like:

```text
┌──────────────────────┐
│ Layer 2              │
│ Added /app           │
├──────────────────────┤
│ Layer 1              │
│ Ubuntu filesystem    │
└──────────────────────┘
```

When Docker looks at the complete image, it combines the layers.

The resulting filesystem appears as:

```text
/
├── app/
├── bin/
├── etc/
├── home/
├── usr/
└── var/
```

**This is the first important concept.**

A layer does not necessarily represent a complete filesystem.

It represents **changes to the filesystem**.

---

# 3. Add another change

Now:

```dockerfile
RUN mkdir /app/logs
```

Another change occurs.

```text
Layer 3
────────────
Added:
/app/logs/
```

Now:

```text
┌──────────────────────┐
│ Layer 3              │
│ Added /app/logs      │
├──────────────────────┤
│ Layer 2              │
│ Added /app           │
├──────────────────────┤
│ Layer 1              │
│ Ubuntu filesystem    │
└──────────────────────┘
```

The final filesystem becomes:

```text
/
├── app/
│   └── logs/
├── bin/
├── etc/
├── home/
├── usr/
└── var/
```

So:

```text
Layer 1
   ↓
Ubuntu

Layer 2
   ↓
/app

Layer 3
   ↓
/app/logs
```

That's what **layering** means.

---

# 4. Now let's use a real application

Suppose you have:

```text
myapp/
├── Dockerfile
├── requirements.txt
└── app.py
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

Let's understand exactly what happens.

---

# 5. First: `FROM`

```dockerfile
FROM python:3.12
```

You are saying:

> "Start my image using the filesystem provided by the Python image."

The Python image itself already contains layers.

Conceptually:

```text
python:3.12

┌────────────────────────┐
│ Python-related files   │
├────────────────────────┤
│ OS filesystem          │
├────────────────────────┤
│ Base filesystem        │
└────────────────────────┘
```

So your image doesn't start from zero.

It starts here:

```text
Your image

┌────────────────────────┐
│                        │
│     Your changes       │
│                        │
├────────────────────────┤
│ Python image layers    │
├────────────────────────┤
│ Python image layers    │
└────────────────────────┘
```

This is extremely important.

---

# 6. `WORKDIR /app`

Next:

```dockerfile
WORKDIR /app
```

Docker establishes `/app` as the working directory.

For the **mental model**, you can think of this as part of the image state/configuration.

But don't get stuck on:

> "`WORKDIR` always creates a filesystem layer."

That's not the important lesson.

The important thing is:

```text
FROM
 ↓
starting image

WORKDIR
 ↓
working-directory configuration

COPY
 ↓
filesystem change

RUN
 ↓
filesystem change

COPY
 ↓
filesystem change

CMD
 ↓
container startup configuration
```

---

# 7. `COPY requirements.txt .`

Now we get to something important.

You have:

```text
requirements.txt
```

on your build context.

Docker executes:

```dockerfile
COPY requirements.txt .
```

Suppose the file contains:

```text
flask==3.1.0
requests==2.32.3
```

Docker adds that file to `/app`.

Now the filesystem contains:

```text
/app/
└── requirements.txt
```

Conceptually:

```text
┌────────────────────────────┐
│ Layer                      │
│                            │
│ /app/requirements.txt      │
├────────────────────────────┤
│ Python base image          │
└────────────────────────────┘
```

This is a filesystem change.

---

# 8. Then `RUN pip install`

Now:

```dockerfile
RUN pip install -r requirements.txt
```

This command executes **during the image build**.

It installs:

```text
Flask
Requests
...
```

into the image filesystem.

So another filesystem change happens.

Conceptually:

```text
┌────────────────────────────┐
│ Layer 4                    │
│ Python packages installed  │
├────────────────────────────┤
│ Layer 3                    │
│ requirements.txt           │
├────────────────────────────┤
│ Python base image          │
└────────────────────────────┘
```

Now the image contains:

```text
/app/
└── requirements.txt

/usr/local/lib/python3.12/
├── flask
├── requests
└── ...
```

This is a major layer.

Why?

Because installing dependencies might take:

```text
10 seconds
30 seconds
2 minutes
```

depending on the application.

---

# 9. Then `COPY app.py`

Now:

```dockerfile
COPY app.py .
```

Docker adds:

```text
/app/app.py
```

Now conceptually:

```text
┌────────────────────────────┐
│ Layer 5                    │
│ /app/app.py                │
├────────────────────────────┤
│ Layer 4                    │
│ Python dependencies        │
├────────────────────────────┤
│ Layer 3                    │
│ requirements.txt           │
├────────────────────────────┤
│ Python base image          │
└────────────────────────────┘
```

That's your image.

---

# 10. Here's the key idea

When you run:

```bash
docker run myapp:1.0
```

Docker doesn't have five completely separate filesystems.

Instead, Docker presents a **combined filesystem view**.

Think:

```text
Layer 5
   ↓
Layer 4
   ↓
Layer 3
   ↓
Base layers
```

Docker combines them.

You see:

```text
/
├── app/
│   ├── requirements.txt
│   └── app.py
│
├── usr/
│   └── local/
│       └── lib/
│           └── python3.12/
│               ├── flask/
│               ├── requests/
│               └── ...
│
├── bin/
├── etc/
└── ...
```

So the application sees **one filesystem**, even though internally it is assembled from layers.

---

# 11. Now the REALLY important part: Why layers?

Imagine you build:

```text
myapp:1.0
```

Today.

Docker creates:

```text
Base Python layers
        ↓
requirements.txt layer
        ↓
pip install layer
        ↓
app.py layer
```

Now tomorrow you change only:

```text
app.py
```

You build again.

Docker asks:

> "Did the earlier parts change?"

Python base?

```text
No
```

`requirements.txt`?

```text
No
```

Dependency installation?

```text
No
```

`app.py`?

```text
YES
```

Therefore Docker can potentially reuse:

```text
Python base
     ↓
requirements.txt
     ↓
pip install
```

and rebuild the changed part.

Conceptually:

```text
OLD BUILD

Python base          ✅
requirements.txt     ✅
pip install          ✅
app.py                ✅


NEW BUILD

Python base          ♻️ reuse
requirements.txt     ♻️ reuse
pip install          ♻️ reuse
app.py                🔨 rebuild
```

**That is why layers matter.**

---

# 12. Without layers

Imagine Docker treated the image as one giant object:

```text
┌───────────────────────────┐
│                           │
│ Python                    │
│ dependencies              │
│ requirements              │
│ application               │
│ everything                │
│                           │
└───────────────────────────┘
```

You change:

```text
app.py
```

Docker would have to rebuild the whole thing.

Conceptually:

```text
Change app.py
     ↓
Rebuild everything
     ↓
Slow
```

With layers:

```text
Change app.py
     ↓
Earlier layers still valid
     ↓
Reuse them
     ↓
Build only necessary later work
     ↓
Fast
```

This is the fundamental reason layers are valuable.

---

# 13. Now understand Dockerfile ordering

This is where your CI/CD concern comes in.

Good:

```dockerfile
COPY requirements.txt .

RUN pip install -r requirements.txt

COPY app.py .
```

Why?

Because these two things change at different frequencies.

Usually:

```text
requirements.txt
        ↓
changes occasionally

app.py
        ↓
changes frequently
```

Therefore we separate them.

Conceptually:

```text
Dependency layer
        ↓
changes rarely
        ↓
cache it

Application layer
        ↓
changes frequently
        ↓
rebuild it
```

---

# 14. Bad ordering

Suppose you write:

```dockerfile
COPY . .

RUN pip install -r requirements.txt
```

Your directory:

```text
myapp/
├── Dockerfile
├── requirements.txt
├── app.py
├── config.py
└── README.md
```

`COPY . .` copies everything.

Now you change only:

```text
README.md
```

The contents of the `COPY . .` instruction change.

That can invalidate the cache for the following:

```dockerfile
RUN pip install -r requirements.txt
```

So Docker may need to run:

```text
pip install
```

again.

Even though:

```text
requirements.txt
```

didn't change.

That's wasteful.

---

# 15. This is the CI/CD connection

Imagine Jenkins builds your application:

```text
Developer
    │
    ▼
Git push
    │
    ▼
Jenkins
    │
    ▼
docker build
```

Your developers push code 50 times per day.

If your Dockerfile is optimized:

```text
requirements unchanged
       ↓
dependency layer cached
       ↓
only application changes
       ↓
fast build
```

If poorly structured:

```text
source changed
       ↓
large COPY invalidated
       ↓
dependency installation runs again
       ↓
slow build
```

Multiply that by:

```text
10 builds/day
50 builds/day
100 builds/day
```

and Dockerfile layer design becomes a real CI/CD performance issue.

---

# 16. Now let's understand "immutable"

This word confused many beginners.

Suppose this exists:

```text
Layer 3

/app/app.py
```

Later you change `app.py`.

Docker doesn't normally go inside the old layer and edit:

```text
Layer 3
```

Instead, the new build produces another filesystem change.

Think:

```text
Old image

Layer C
app.py version 1
     ↓
Layer B
dependencies
     ↓
Layer A
Python
```

New image:

```text
Layer D
app.py version 2
     ↓
Layer B
dependencies
     ↓
Layer A
Python
```

Notice:

```text
Layer A → reused
Layer B → reused
Layer C → not needed by new image
Layer D → new
```

That's the important meaning of **immutable layers**.

---

# 17. What about deleting a file?

This is another place where layers become interesting.

Suppose:

```dockerfile
RUN echo "secret" > /tmp/password
```

A layer contains:

```text
/tmp/password
```

Then:

```dockerfile
RUN rm /tmp/password
```

The new filesystem view no longer shows:

```text
/tmp/password
```

But the previous layer may still contain the data.

Think:

```text
Layer 2
────────────────
/tmp/password
"secret"


Layer 3
────────────────
Delete /tmp/password
```

Final filesystem:

```text
/tmp/password
     ↓
not visible
```

But:

```text
old layer
     ↓
may still contain it
```

Therefore:

> **Deleting a secret in a later Dockerfile instruction does not make the secret safe.**

This is why you don't do:

```dockerfile
RUN echo "PASSWORD=secret" > /tmp/config
RUN rm /tmp/config
```

You should avoid putting secrets into normal image layers in the first place.

---

# 18. Image layers vs container layer

This distinction is extremely important.

Suppose your image is:

```text
Image

Layer 4
app.py

Layer 3
dependencies

Layer 2
requirements

Layer 1
Python
```

These image layers are effectively read-only.

Now you run:

```bash
docker run myapp:1.0
```

Docker creates a container.

Conceptually:

```text
Container

┌─────────────────────────────┐
│ Writable container layer    │ ← changes here
├─────────────────────────────┤
│ Image Layer 4               │
├─────────────────────────────┤
│ Image Layer 3               │
├─────────────────────────────┤
│ Image Layer 2               │
├─────────────────────────────┤
│ Image Layer 1               │
└─────────────────────────────┘
```

Suppose the application creates:

```text
/app/output.log
```

That change goes into the **container's writable layer**, not into the original image layer.

This is why:

```text
Image
```

and

```text
Container
```

are different concepts.

We'll study this properly in **Part 6 — Container Filesystem**.

---

# 19. The easiest analogy

Think of Docker layers like **transparent sheets**.

Imagine:

### Sheet 1

```text
Ubuntu filesystem
```

Put another transparent sheet on top:

### Sheet 2

```text
Add /app
```

Another:

### Sheet 3

```text
Add Python dependencies
```

Another:

### Sheet 4

```text
Add app.py
```

Put all sheets together:

```text
       Sheet 4
     app.py
───────────────
       Sheet 3
   dependencies
───────────────
       Sheet 2
       /app
───────────────
       Sheet 1
       Ubuntu
```

Looking from the top, you see one complete filesystem.

But internally, it's built from multiple layers.

That's probably the simplest mental model to keep.

---

# 20. One more important correction

You showed this model:

```text
FROM
 ↓
Layer

WORKDIR
 ↓
Layer

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

Don't memorize this literally.

A better model is:

```text
Dockerfile
     │
     ├── FROM
     │     ↓
     │   existing image/layers
     │
     ├── COPY
     │     ↓
     │   filesystem change
     │
     ├── RUN
     │     ↓
     │   filesystem change
     │
     ├── COPY
     │     ↓
     │   filesystem change
     │
     └── CMD
           ↓
       image configuration
```

Modern Docker/BuildKit makes the actual implementation more sophisticated than:

```text
one instruction = one physical layer
```

For learning Docker and CI/CD, think primarily in terms of:

> **filesystem changes + cacheable build steps + final image layers**

---

# 21. Let's look at a real build

Create:

```text
layer-demo/
├── Dockerfile
├── requirements.txt
└── app.py
```

### Dockerfile

```dockerfile
FROM python:3.12

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY app.py .

CMD ["python", "app.py"]
```

Build:

```bash
docker build -t layer-demo:1.0 .
```

Then:

```bash
docker image history layer-demo:1.0
```

You'll see entries representing the image's construction/history.

Also:

```bash
docker image inspect layer-demo:1.0
```

This lets you inspect image metadata and its root filesystem information.

---

# 22. The whole concept in one picture

This is the model I want you to remember:

```text
                     Dockerfile
                         │
                         ▼
                ┌─────────────────┐
                │ FROM python     │
                └────────┬────────┘
                         │
                         ▼
                 Existing layers
                         │
                         ▼
                ┌─────────────────┐
                │ COPY requirements│
                └────────┬────────┘
                         │
                         ▼
                  filesystem change
                         │
                         ▼
                ┌─────────────────┐
                │ RUN pip install │
                └────────┬────────┘
                         │
                         ▼
                  filesystem change
                         │
                         ▼
                ┌─────────────────┐
                │ COPY app.py     │
                └────────┬────────┘
                         │
                         ▼
                  filesystem change
                         │
                         ▼
                  ┌─────────────┐
                  │ Docker Image│
                  └──────┬──────┘
                         │
                         ▼
                    docker run
                         │
                         ▼
                  ┌─────────────┐
                  │  Container  │
                  ├─────────────┤
                  │ Writable    │
                  │ layer       │
                  ├─────────────┤
                  │ Image layer │
                  ├─────────────┤
                  │ Image layer │
                  ├─────────────┤
                  │ Base layers │
                  └─────────────┘
```

---

# 23. The 5 things you should understand before moving on

If these five points are clear, you've understood Docker layers:

### 1. An image isn't one giant filesystem

It is assembled from layers.

### 2. A layer represents filesystem changes

For example:

```text
Add file
Install package
Create directory
Delete file
```

### 3. Layers are reusable

If an earlier part hasn't changed, Docker can potentially reuse it.

### 4. This creates the build-cache advantage

```text
unchanged work
      ↓
reuse
      ↓
less rebuilding
      ↓
faster CI
```

### 5. Dockerfile ordering matters

Prefer:

```dockerfile
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
```

over:

```dockerfile
COPY . .
RUN pip install -r requirements.txt
```

when you want dependency installation to remain cacheable when application source changes.

---

## The single sentence I want you to remember

> **A Docker image is a stack of filesystem changes built on top of a base image; because unchanged layers can be reused, Docker can avoid repeating expensive build work.**

And **that** is why image layers matter to your CI/CD learning.

