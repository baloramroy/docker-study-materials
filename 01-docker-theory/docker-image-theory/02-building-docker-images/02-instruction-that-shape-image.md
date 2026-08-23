# Phase 2 — Build Your First Docker Image

## Part 2 — Instructions That Shape the Image

### Where We Are

In Part 1, we established what we're building:

```text
Application
    ↓
Dockerfile
    ↓
docker build
    ↓
Docker Image
```

We now have our application:

```text
my-app/
└── app.py
```

with:

```python
print("Hello from Docker!")
```

Now we're ready to write the first part of the Dockerfile.

The three instructions in this part are:

```text
FROM
WORKDIR
COPY
```

They answer three basic questions:

```text
FROM
 ↓
Where does my image start?

WORKDIR
 ↓
Where will my application live/work?

COPY
 ↓
How do I put my application files there?
```

---

# 1. A Quick Look at Dockerfile Instructions

A Dockerfile is made up of instructions.

For example:

```dockerfile
FROM python:3.12
WORKDIR /app
COPY app.py .
```

Each instruction tells Docker to perform a particular operation while constructing the image.

For this part, we're focusing only on instructions that establish the image and put our application into it.

We'll deal with commands that execute things later when we learn `RUN`.

---

# 2. `FROM` — Choose Where the Image Starts

Let's start with:

```dockerfile
FROM python:3.12
```

`FROM` tells Docker:

> **Use this image as the starting point for my new image.**

This is the first instruction in a normal Dockerfile.

---

## Why Do We Need a Starting Image?

Our application requires Python:

```python
print("Hello from Docker!")
```

We could theoretically construct an environment from almost nothing and install everything ourselves.

That would mean dealing with things such as:

```text
Operating system
    ↓
System libraries
    ↓
Python
    ↓
Python configuration
    ↓
Application
```

That's unnecessary.

Instead, we can use an existing Python image:

```text
python:3.12
     │
     │ starting point
     ▼
our Dockerfile
     │
     ▼
my-app:1.0
```

So:

```dockerfile
FROM python:3.12
```

basically says:

> "Start my image from the existing `python:3.12` image."

---

# 3. What Is `python:3.12`?

You already learned image tags in Phase 1, so here's only the relevant callback:

> **Phase 1 callback:** `python:3.12` follows the `image-name:tag` format. The `3.12` identifies the requested Python image variant/tag.

So:

```text
python:3.12
│      │
│      └── tag
└───────── image name
```

Docker needs to obtain that base image if it isn't already available locally.

Conceptually:

```text
Dockerfile
    │
    │ FROM python:3.12
    ▼
python:3.12
    │
    │ becomes starting point
    ▼
our image
```

---

# 4. What Does `FROM` Give Us?

After:

```dockerfile
FROM python:3.12
```

our image is no longer an empty concept.

It starts from the filesystem and environment provided by the selected base image.

Think of it like this:

```text
Before FROM:

      Nothing useful for our app yet


After FROM:

Docker Image
├── Python environment
├── Base filesystem
└── Base image contents
```

We haven't added `app.py` yet.

We've simply established the **starting point**.

---

# 5. `WORKDIR` — Choose the Working Directory

Now we add:

```dockerfile
WORKDIR /app
```

This tells Docker:

> **Set `/app` as the working directory for subsequent Dockerfile instructions and for the container's runtime process.**

Think of a normal Linux shell:

```bash
cd /app
```

After doing that, commands execute relative to `/app`.

`WORKDIR` gives Docker the same basic concept without requiring us to repeatedly use `cd`.

---

# 6. Why `/app`?

There's nothing magical about the name `/app`.

We could use:

```text
/app
/myapplication
/opt/my-app
```

or another appropriate directory.

We're choosing:

```dockerfile
WORKDIR /app
```

because `/app` is a common and simple convention for application files.

So our image now conceptually looks like:

```text
Docker Image
│
├── Base image contents
│
└── /app
```

The directory becomes the working location for the instructions that follow.

---

# 7. `WORKDIR` Can Create the Directory

One useful behavior:

If `/app` doesn't already exist, Docker creates it.

So:

```dockerfile
WORKDIR /app
```

doesn't require us to first execute:

```bash
mkdir /app
```

We don't need:

```dockerfile
RUN mkdir /app
WORKDIR /app
```

The `WORKDIR` instruction itself handles the directory setup.

That's one reason using `WORKDIR` is preferable to manually creating a directory and then changing into it.

---

# 8. Why `WORKDIR` Matters for the Next Instruction

Now consider:

```dockerfile
WORKDIR /app

COPY app.py .
```

The `.` in the `COPY` destination refers to the current working directory.

Our current working directory is:

```text
/app
```

Therefore:

```dockerfile
COPY app.py .
```

puts the file at:

```text
/app/app.py
```

The sequence is:

```text
WORKDIR /app
      ↓
Current working directory = /app
      ↓
COPY app.py .
      ↓
/app/app.py
```

This is why `WORKDIR` and `COPY` make sense together.

---

# 9. `COPY` — Put Application Files Into the Image

Now we reach:

```dockerfile
COPY app.py .
```

`COPY` tells Docker:

> **Copy files from the build context into the image.**

The general form is:

```dockerfile
COPY <source> <destination>
```

Our example:

```dockerfile
COPY app.py .
```

has:

```text
source       destination
  │               │
  ▼               ▼
app.py             .
```

---

# 10. Where Does `app.py` Come From?

Here's our project:

```text
my-app/
├── app.py
└── Dockerfile
```

The build command will eventually be:

```bash
docker build -t my-app:1.0 .
```

The `.` gives Docker the current directory as the build context.

**Phase 1 callback:** we already established what the build context is. Here we're simply applying that concept: `COPY` can use files available within that build context.

So:

```text
Build Context
│
├── app.py
└── Dockerfile
```

Then:

```dockerfile
COPY app.py .
```

copies `app.py` into the image.

---

# 11. What Does the Destination `.` Mean?

This is the part that often confuses beginners.

We have:

```dockerfile
WORKDIR /app
```

Then:

```dockerfile
COPY app.py .
```

The destination:

```text
.
```

means the current working directory.

And the current working directory is:

```text
/app
```

Therefore:

```text
app.py
  │
  │ COPY
  ▼
/app/app.py
```

So after these instructions, our image contains:

```text
Docker Image
│
├── base image contents
│
└── app/
    └── app.py
```

---

# 12. The Three Instructions Together

Now let's combine everything we've learned:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY app.py .
```

Read this Dockerfile as a sequence of decisions:

### Step 1

```dockerfile
FROM python:3.12
```

> Start with the Python 3.12 image.

### Step 2

```dockerfile
WORKDIR /app
```

> Make `/app` the working directory.

### Step 3

```dockerfile
COPY app.py .
```

> Copy `app.py` from the build context into `/app`.

The resulting structure is conceptually:

```text
Base Image
    │
    ▼
Python environment
    │
    │ WORKDIR /app
    ▼
/app
    │
    │ COPY app.py .
    ▼
/app/app.py
```

---

# 13. A Very Important Distinction: Host vs Image

At this point, we should clearly separate two filesystems.

### On our host:

```text
my-app/
├── app.py
└── Dockerfile
```

### Inside the image:

```text
/
├── ...
└── app/
    └── app.py
```

The Dockerfile instruction:

```dockerfile
COPY app.py .
```

is moving the file conceptually from:

```text
Host build context
       │
       ▼
   app.py
       │
       │ COPY
       ▼
Image filesystem
       │
       ▼
 /app/app.py
```

The original host file isn't moved or deleted.

It's **copied** into the image.

---

# 14. Why `COPY` Doesn't Mean "Mount"

This is worth clarifying because we'll deal with volumes and bind mounts later.

`COPY` puts a copy of the file **into the image during the build**.

It does not create a live connection between the host and container.

Conceptually:

```text
Host
app.py
  │
  │ COPY during build
  ▼
Image
/app/app.py
```

After the image has been built, the image has its own copy.

If you later change the host's:

```text
app.py
```

the already-built image does not automatically change.

You would need to build a new image.

This is fundamentally different from a bind mount, which we'll deal with later.

---

# 15. What Have We Built So Far?

Our Dockerfile currently contains:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY app.py .
```

We can visualize the image-building process:

```text
             Dockerfile
                 │
                 ▼
        FROM python:3.12
                 │
                 ▼
       Python base image
                 │
                 ▼
          WORKDIR /app
                 │
                 ▼
             /app/
                 │
                 ▼
         COPY app.py .
                 │
                 ▼
          /app/app.py
```

At this point, we've established an environment and placed our application inside it.

But there's one problem:

> **We haven't told Docker what to do with `app.py` when a container starts.**

That's coming in the next part.

---

# 16. A Short Look at the Image Construction

There's also a useful connection to the image layers you learned in Phase 1.

**Phase 1 callback:** Docker images are built from layers, and filesystem-changing Dockerfile instructions can contribute to those layers.

So as Docker processes our Dockerfile:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY app.py .
```

we can think about the image evolving:

```text
python:3.12
     │
     ▼
Base image
     │
     ▼
Working directory established
     │
     ▼
app.py added
     │
     ▼
Resulting image
```

We don't need to re-learn image layers here. The important new idea is:

> **Dockerfile instructions progressively construct the image.**

---

# 17. One More Look at `FROM`

There's a subtle but important distinction between these ideas:

```text
FROM
```

and:

```text
COPY
```

`FROM` establishes **where our image starts**.

`COPY` adds **our application files**.

So:

```text
FROM
 ↓
Existing environment

COPY
 ↓
Our application
```

That's the basic pattern we'll keep building on.

---

# Part 2 Checkpoint

Before moving on, you should now be able to explain the following Dockerfile:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY app.py .
```

### `FROM`

Where does the image start?

```text
python:3.12
```

### `WORKDIR`

Where is our working directory?

```text
/app
```

### `COPY`

What happens to `app.py`?

```text
Build context/app.py
        ↓
Image
        ↓
/app/app.py
```

---

## The Mental Model

Keep this picture:

```text
          FROM
           │
           ▼
    python:3.12
           │
           ▼
       WORKDIR
        /app
           │
           ▼
         COPY
       app.py
           │
           ▼
     /app/app.py
```

So these three instructions answer:

```text
FROM
→ Where do I start?

WORKDIR
→ Where do I work?

COPY
→ What application files do I put there?
```

**Part 2 is complete.**

We now have an image that conceptually contains our Python environment and our application.

But we still haven't answered a critical question:

> **What is the difference between something Docker does while BUILDING the image and something Docker does when RUNNING a container?**

That's the central topic of **Part 3 — Build-Time vs Runtime**, where we'll introduce `RUN` and `CMD`.
