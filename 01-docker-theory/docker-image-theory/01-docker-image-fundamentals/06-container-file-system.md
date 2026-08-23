# Phase 1 — Part 6: Container Filesystem

## Where we are

```text
Phase 1 — Docker Fundamentals

01. Container vs Image       ✅
02. Dockerfile               ✅
03. Build Context            ✅
04. Docker Image Layers      ✅
05. Docker Image Registry    ✅
06. Container Filesystem     ← NOW
07. Base Images
08. Image Tags
```

This part connects directly to what we learned about image layers.

The key question is:

> **When a container starts from an image, where does its filesystem come from, and what happens when the container changes files?**

---

# 1. Start with the basic concept

Let's start with something very simple.

Suppose you have this image:

```text
my-app:1.0
```

Inside that image there might be:

```text
/
├── bin/
├── etc/
├── home/
├── usr/
├── var/
└── app/
    ├── app.py
    └── requirements.txt
```

When you create a container from this image:

```bash
docker run my-app:1.0
```

the container gets a filesystem based on the image.

So conceptually:

```text
Image
  |
  | docker run
  v
Container
  |
  +-- /
      ├── bin/
      ├── etc/
      ├── usr/
      ├── var/
      └── app/
```

But there is a very important detail:

> **The container does not simply modify the image itself.**

Instead, Docker creates a **writable layer on top of the image's read-only layers**.

That is the heart of this lesson.

---

# 2. The Read-Only Image Layers

Remember Part 4.

An image consists of layers:

```text
Image
┌──────────────────────────┐
│ Layer 3                  │
├──────────────────────────┤
│ Layer 2                  │
├──────────────────────────┤
│ Layer 1                  │
└──────────────────────────┘
```

These image layers are treated as **read-only** when used by a running container.

But containers need to be able to modify files.

For example, an application may need to:

```text
create a log
update a temporary file
write application data
modify configuration
create a cache
```

Docker therefore adds a writable layer on top.

---

# 3. The Container Writable Layer

When a container is created, the basic structure is:

```text
Container
┌──────────────────────────────┐
│ Writable Container Layer     │
├──────────────────────────────┤
│ Image Layer                  │
├──────────────────────────────┤
│ Image Layer                  │
├──────────────────────────────┤
│ Image Layer                  │
└──────────────────────────────┘
```

The image layers remain read-only.

The top layer belongs to the container and is writable.

This is the key filesystem model:

> **Image = read-only layers**
> **Container = image layers + writable container layer**

---

# 4. What Does the Container Actually See?

Although there are multiple layers underneath, the application normally sees them as **one unified filesystem**.

For example:

```text
Container
│
├── /bin
├── /etc
├── /usr
├── /var
└── /app
```

The application doesn't normally need to know:

```text
/app came from Layer 3
/etc came from Layer 2
/bin came from Layer 1
```

Docker's storage system presents a unified filesystem view.

Conceptually:

```text
Image Layers
     +
Writable Layer
     │
     ▼
Unified Container Filesystem
```

---

# 5. Reading a File

Suppose the image contains:

```text
/app/app.py
```

When the container starts:

```bash
docker run my-app
```

the container can read:

```bash
cat /app/app.py
```

If the application only reads the file, Docker doesn't need to create a new copy in the writable layer.

Conceptually:

```text
Read /app/app.py
       │
       ▼
Image layer
```

The existing image data can be reused.

---

# 6. What Happens When the Container Modifies a File?

Now suppose:

```text
/app/config.txt
```

already exists in the image.

Inside the running container:

```bash
echo "new value" > /app/config.txt
```

The original image layer isn't modified.

Instead, Docker's storage system handles the modification through the writable container layer.

Conceptually:

```text
Before:

Writable Layer
     │
     │ empty
     ▼
Image Layer
└── /app/config.txt


After modification:

Writable Container Layer
└── modified /app/config.txt

Image Layer
└── original /app/config.txt
```

The container sees the modified version.

The original image remains unchanged.

---

# 7. This Is Why Containers Don't Modify the Image

Suppose you run:

```bash
docker run -it ubuntu bash
```

and inside the container:

```bash
touch /tmp/test.txt
```

You have modified the **container**, not the Ubuntu image.

If you then remove the container:

```bash
docker rm <container>
```

the modification disappears.

The image is still unchanged.

So:

```text
Image
  │
  ├── Container A
  │      └── changes
  │
  └── Container B
         └── changes
```

Container A's changes don't become part of the image.

---

# 8. Hands-on Exercise — Container Changes

Let's see this directly.

Run:

```bash
docker run -it --name filesystem-demo alpine sh
```

Inside the container:

```bash
echo "Hello Docker" > /tmp/test.txt
```

Check it:

```bash
cat /tmp/test.txt
```

You should get:

```text
Hello Docker
```

Now exit:

```bash
exit
```

The container has stopped, but still exists.

Check:

```bash
docker ps -a
```

Start it again:

```bash
docker start filesystem-demo
```

Then:

```bash
docker exec filesystem-demo cat /tmp/test.txt
```

You should still get:

```text
Hello Docker
```

Why?

Because the container still exists, including its writable layer.

---

# 9. Remove the Container

Now remove it:

```bash
docker rm -f filesystem-demo
```

Create a completely new container:

```bash
docker run --rm alpine cat /tmp/test.txt
```

You should get an error because `/tmp/test.txt` isn't present.

Why?

Because:

```text
Old Container
└── writable layer
    └── /tmp/test.txt
```

was deleted when the container was removed.

The Alpine image itself never contained:

```text
/tmp/test.txt
```

---

# 10. The Important Difference

This gives us a very important lifecycle:

```text
docker run
     │
     ▼
Container created
     │
     ▼
Writable layer created
     │
     ▼
Application modifies files
     │
     ▼
Changes stored in container layer
     │
     ▼
Container removed
     │
     ▼
Writable layer removed
```

Therefore:

> **Data written only to the container's writable layer is tied to that container's lifecycle.**

---

# 11. Container Stop vs Container Remove

This distinction is important.

### Stop

```bash
docker stop my-container
```

The container stops running, but the container still exists.

Its writable layer remains.

```text
Container
└── Writable Layer
    └── data still exists
```

If you start it again:

```bash
docker start my-container
```

the data is still there.

### Remove

```bash
docker rm my-container
```

The container and its writable layer are removed.

```text
Container
└── Writable Layer
    └── removed
```

So:

```text
STOP
  ↓
Container remains
  ↓
Writable data remains


REMOVE
  ↓
Container disappears
  ↓
Writable layer disappears
```

---

# 12. Why This Matters for Applications

Imagine a database running inside a container:

```text
MySQL Container
└── /var/lib/mysql/
```

If the database stores everything only inside the container's writable layer:

```text
Container
└── Writable Layer
    └── database files
```

then removing the container can remove the database data.

That is obviously undesirable.

This leads to an important Docker concept:

> **Persistent application data should generally not depend solely on the container's writable layer.**

This is why Docker provides **volumes** and other storage mechanisms.

We won't go deeply into volumes here because they're outside the current Phase 1 roadmap.

For now, just understand **why persistent storage mechanisms are necessary**.

---

# 13. Container Filesystem Is Not the Same as Host Filesystem

Another common misunderstanding:

> "If I create `/app/test.txt` inside the container, I created it on the host."

Not necessarily.

Without a mounted storage mechanism:

```text
Host Filesystem
       │
       │ isolated
       ▼
Container Filesystem
       │
       └── /app/test.txt
```

The file belongs to the container's filesystem.

It doesn't automatically appear somewhere like:

```text
/home/user/app/test.txt
```

on the host.

This filesystem isolation is one of the fundamental properties of containers.

---

# 14. Containers Share the Host Kernel

There is one important clarification.

Container filesystem isolation does **not** mean a container has its own complete operating-system kernel.

A simplified model is:

```text
Host
┌──────────────────────────────┐
│ Linux Kernel                 │
│                              │
│   ┌─────────┐  ┌─────────┐  │
│   │Container│  │Container│  │
│   │Filesystem│ │Filesystem│ │
│   └─────────┘  └─────────┘  │
└──────────────────────────────┘
```

Containers have isolated:

* filesystem views
* processes
* networking
* other namespaces/resources

but they normally share the host's kernel.

This is one of the fundamental differences between containers and traditional virtual machines.

We don't need to go deeper into Linux namespaces here.

---

# 15. Hands-on Exercise — See the Container Filesystem

Run:

```bash
docker run -it --name fs-demo alpine sh
```

Inside:

```bash
ls /
```

You'll see directories such as:

```text
bin
dev
etc
home
proc
root
sys
tmp
usr
var
```

Now:

```bash
mkdir /mydata
echo "container data" > /mydata/data.txt
```

Check:

```bash
cat /mydata/data.txt
```

Exit:

```bash
exit
```

Now start the same container:

```bash
docker start fs-demo
```

Check:

```bash
docker exec fs-demo cat /mydata/data.txt
```

The data remains because the container still exists.

Now:

```bash
docker rm -f fs-demo
```

Create a new container:

```bash
docker run --rm alpine ls /mydata
```

It should report that `/mydata` doesn't exist.

That demonstrates the lifecycle of the writable container layer.

---

# 16. How This Connects to Image Layers

We can now combine Part 4 and Part 6.

An image:

```text
┌──────────────────────┐
│ Image Layer 3        │
├──────────────────────┤
│ Image Layer 2        │
├──────────────────────┤
│ Image Layer 1        │
└──────────────────────┘
```

A running container:

```text
┌──────────────────────────────┐
│ Writable Container Layer     │
├──────────────────────────────┤
│ Image Layer 3                │
├──────────────────────────────┤
│ Image Layer 2                │
├──────────────────────────────┤
│ Image Layer 1                │
└──────────────────────────────┘
```

So the relationship is:

```text
Docker Image
     │
     │ docker run
     ▼
Container
     │
     ├── Image layers (read-only)
     │
     └── Writable container layer
```

This is one of the most important Docker filesystem mental models.

---

# 17. Common Mistakes

### Mistake 1 — Thinking a container modifies the image

Incorrect:

```text
Container changes
      ↓
Image changes
```

Correct:

```text
Container changes
      ↓
Container writable layer
```

The original image remains unchanged.

---

### Mistake 2 — Thinking stopped containers lose their data

Incorrect:

```text
docker stop
    ↓
data deleted
```

Correct:

```text
docker stop
    ↓
container still exists
    ↓
writable data remains
```

Data is normally lost when the container itself is removed, assuming it wasn't stored in persistent storage.

---

### Mistake 3 — Treating the writable layer as persistent storage

The writable layer exists for the container's lifecycle.

For important application data, use appropriate persistent storage such as Docker volumes or external storage.

---

# 18. What You Should Know Now

You should now understand:

* A container gets its initial filesystem from the image.
* Image layers are read-only.
* A container gets a writable layer on top of those image layers.
* The container sees all of this as one unified filesystem.
* Modifications happen in the container's writable layer rather than changing the image.
* Stopping a container does not remove its writable layer.
* Removing the container normally removes that writable layer.
* Container filesystem data is therefore not automatically persistent.
* Persistent application data requires an appropriate storage mechanism.
* Container filesystem isolation is separate from the host filesystem.

The key model is:

```text
              Docker Image
        ┌──────────────────────┐
        │ Image Layer 3        │
        │ Image Layer 2        │
        │ Image Layer 1        │
        └──────────────────────┘
                  │
                  │ docker run
                  ▼
        ┌──────────────────────┐
        │ Writable Layer       │
        ├──────────────────────┤
        │ Image Layer 3        │
        │ Image Layer 2        │
        │ Image Layer 1        │
        └──────────────────────┘
                  │
                  ▼
          Container Filesystem
```

---

# Phase 1 Progress

```text
Phase 1 — Docker Fundamentals

01. Container vs Image       ✅
02. Dockerfile               ✅
03. Build Context            ✅
04. Docker Image Layers      ✅
05. Docker Image Registry    ✅
06. Container Filesystem     ✅
07. Base Images              ← NEXT
08. Image Tags
```

We've now completed the filesystem relationship:

```text
Dockerfile
    ↓
Build Context
    ↓
Image Layers
    ↓
Docker Image
    ↓
docker run
    ↓
Container
    ↓
Writable Container Layer
```

**Next: Part 7 — Base Images.**

There we'll answer an important question that has already appeared several times:

> **When we write `FROM alpine`, `FROM ubuntu`, or `FROM python:3.12`, what exactly are we starting our image from, and how does that base image become part of our final image?**
