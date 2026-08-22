# Phase 1 — Part 6: Container Filesystem

We are now moving to:

```text
01. Container vs Image       ✅
02. Dockerfile               ✅
03. Build context            ✅
04. Docker image layers      ✅
05. Docker image registry    ✅
06. Container filesystem     ← NOW
07. Base images
08. Image tags
```

This topic is extremely important because it connects the concepts we've already learned:

```text
Dockerfile
    ↓
Build context
    ↓
Image layers
    ↓
Image registry
    ↓
Image
    ↓
Container filesystem   ← NOW
```

The key question we want to answer is:

> **When Docker creates a container from an image, what filesystem does the container actually see?**

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

# 2. Connect this to image layers

In Step 4, we learned that an image is composed of layers.

For example:

```text
Image
│
├── Layer 4 — application files
├── Layer 3 — dependencies
├── Layer 2 — runtime
└── Layer 1 — base filesystem
```

These image layers are effectively **read-only** when the image is used by a container.

When Docker creates a container, it adds a new writable layer:

```text
Container
│
├── Writable container layer
│
├── Image Layer 4
├── Image Layer 3
├── Image Layer 2
└── Image Layer 1
```

This is the fundamental filesystem model.

---

# 3. The mental model

Think of the image as a **read-only foundation**.

Then think of the container as:

```text
Read-only image
       +
Writable container layer
       =
Container filesystem
```

Or visually:

```text
              Container filesystem
                     │
          ┌──────────┴──────────┐
          │                     │
   Writable layer         Image layers
          │                     │
    container changes       read-only
```

This is why two containers can be created from the same image without modifying the image itself.

---

# 4. A concrete example

Suppose we have this image:

```text
my-app:1.0
```

Its filesystem contains:

```text
/app/app.py
/app/config.txt
```

Now we create two containers:

```bash
docker run --name app1 my-app:1.0
docker run --name app2 my-app:1.0
```

We now have:

```text
                 my-app:1.0
                 Image layers
                 /          \
                /            \
             app1            app2
              |               |
        writable layer   writable layer
```

Both containers start with the same filesystem content.

But their writable changes are independent.

---

# 5. Let's prove it

Suppose we create a container:

```bash
docker run -it --name test1 ubuntu:24.04 bash
```

Inside the container:

```bash
touch /hello.txt
```

Now:

```bash
ls /
```

will show:

```text
hello.txt
```

We modified the container's filesystem.

But we did **not modify the `ubuntu:24.04` image**.

If we create another container:

```bash
docker run -it --name test2 ubuntu:24.04 bash
```

and run:

```bash
ls /
```

we won't find:

```text
hello.txt
```

Why?

Because:

```text
test1
   ↓
its own writable layer
   ↓
hello.txt
```

while:

```text
test2
   ↓
different writable layer
   ↓
no hello.txt
```

Both containers share the same underlying image layers, but each gets its own writable layer.

---

# 6. This is why containers are isolated

Suppose:

```text
ubuntu:24.04
      |
      +----------+
      |          |
      v          v
   Container A Container B
      |          |
 writable      writable
 layer A       layer B
```

Container A creates:

```text
/data/a.txt
```

Container B creates:

```text
/data/b.txt
```

You effectively have:

```text
Container A
    /data/a.txt

Container B
    /data/b.txt
```

They don't automatically see each other's filesystem changes.

This is part of container isolation.

---

# 7. What does the container actually see?

This is an important point.

Inside a container, you normally see a normal Linux filesystem:

```bash
ls /
```

You might see:

```text
bin
dev
etc
home
lib
lib64
media
mnt
opt
proc
root
run
sbin
sys
tmp
usr
var
```

It looks like a normal Linux filesystem.

That's because, from the application's perspective, it **is interacting with a filesystem namespace that looks like its own root filesystem**.

For example:

```bash
cd /app
ls
```

might show:

```text
app.py
requirements.txt
```

The application doesn't need to know:

> "I'm actually using image layers plus a writable container layer."

Docker/container runtime handles that underneath.

---

# 8. The root filesystem

Inside a container, `/` is the root of that container's filesystem view.

For example:

```text
/
├── etc
├── usr
├── var
├── tmp
└── app
```

This is called the container's **root filesystem** or **rootfs** in many technical contexts.

An important beginner misconception is:

> `/` inside the container is not simply the same `/` as the host.

For example:

```text
HOST
/
├── home
├── etc
├── var
└── ...

CONTAINER
/
├── etc
├── usr
├── var
└── app
```

The container has its own filesystem view.

---

# 9. Container filesystem vs host filesystem

Imagine your Linux host has:

```text
/home/user/project
```

Inside the container, you might have:

```text
/app
```

These are not automatically the same directory.

Without a mount:

```text
Host
/home/user/project
       |
       X
       |
Container
/app
```

They are separate.

If you explicitly mount the host directory:

```bash
docker run \
  -v /home/user/project:/app \
  my-app:1.0
```

then:

```text
Host
/home/user/project
        |
        | mount
        v
Container
/app
```

Now `/app` inside the container corresponds to the host directory.

This is a major concept that we'll eventually connect to **volumes and bind mounts**.

For now, remember:

> **A container filesystem is isolated from the host filesystem unless something is explicitly mounted/shared.**

---

# 10. What happens when a container writes a file?

This is where the writable layer becomes important.

Suppose the image contains:

```text
/app/config.txt
```

The image layer contains:

```text
config.txt
```

Now the application runs:

```text
echo "new configuration" > /app/config.txt
```

What happens?

Docker doesn't go back and modify the original image layer.

Instead, the change is handled through the container's writable layer.

Conceptually:

```text
Image layer
/app/config.txt
       │
       │ original
       ▼
Container writable layer
/app/config.txt
       │
       │ modified version
       ▼
Container sees:
"new configuration"
```

The original image remains unchanged.

---

# 11. Copy-on-write

This behavior is commonly described using the concept **copy-on-write (CoW)**.

The basic idea:

> **Containers can share the underlying read-only image data, while changes are stored separately in the container's writable layer.**

Imagine two containers:

```text
             Image
              │
       ┌──────┴──────┐
       │             │
       v             v
 Container A     Container B
 writable A      writable B
```

Both can use the same image data.

If Container A changes something:

```text
Container A
     |
     +-- changed data
```

Container B isn't automatically affected.

This is one of the reasons containers can be lightweight compared with making a complete copy of an entire operating-system filesystem for every container.

---

# 12. What happens when the container is deleted?

This is extremely important.

Suppose:

```bash
docker run --name test1 ubuntu:24.04
```

Inside it:

```bash
touch /important.txt
```

Now:

```text
Container writable layer
        |
        +-- important.txt
```

If you remove the container:

```bash
docker rm test1
```

the container's writable layer is removed along with the container.

Therefore:

```text
important.txt
```

is gone.

The image is still there:

```text
ubuntu:24.04
```

but your container-specific filesystem change is gone.

---

# 13. This leads to a very important rule

> **The writable layer of a container is temporary storage.**

If your application writes important data directly into the container filesystem:

```text
Container
   |
   +-- database files
   +-- uploaded files
   +-- logs
```

and the container is deleted:

```text
Container
   ↓
deleted
   ↓
writable layer deleted
```

your data may disappear.

This is why Docker provides **persistent storage mechanisms** such as:

* volumes
* bind mounts
* other storage mechanisms

We'll study those later.

For now:

```text
Container filesystem
        ≠
Persistent application storage
```

---

# 14. A very common beginner misconception

A beginner might think:

> "If I install something inside a running container, it becomes part of the Docker image."

No.

Suppose:

```bash
docker run -it ubuntu:24.04 bash
```

Then:

```bash
apt update
apt install nginx
```

Now nginx exists inside that running container.

But you have **not changed `ubuntu:24.04`**.

The image remains the same.

You changed the container's writable layer.

Conceptually:

```text
ubuntu:24.04
     |
     +-- read-only image
             |
             +-- container writable layer
                    |
                    +-- nginx
```

If you delete the container:

```bash
docker rm <container>
```

the installed nginx disappears with it.

---

# 15. Then how does software become part of an image?

Through the image build process.

For example:

```dockerfile
FROM ubuntu:24.04

RUN apt-get update && \
    apt-get install -y nginx
```

Then:

```bash
docker build -t my-nginx:1.0 .
```

Now the installation becomes part of an **image layer**.

Conceptually:

```text
Dockerfile
   |
   v
RUN apt install nginx
   |
   v
Image layer
   |
   v
my-nginx:1.0
```

Then every new container created from that image starts with nginx already present.

This connects our topics together:

```text
Dockerfile
    ↓
Build
    ↓
Image layer
    ↓
Image
    ↓
Container
    ↓
Writable layer
```

---

# 16. Image filesystem vs container filesystem

This distinction is worth memorizing.

### Image

```text
Image
├── Layer 1  read-only
├── Layer 2  read-only
├── Layer 3  read-only
└── Layer 4  read-only
```

### Container

```text
Container
├── Writable layer
├── Image Layer 4
├── Image Layer 3
├── Image Layer 2
└── Image Layer 1
```

So:

> **An image is a reusable read-only template. A container gets a writable layer on top of that image.**

---

# 17. Multiple containers from one image

This is one of Docker's most important design characteristics.

Suppose:

```text
my-app:1.0
```

is your image.

You create:

```bash
docker run --name app1 my-app:1.0
docker run --name app2 my-app:1.0
docker run --name app3 my-app:1.0
```

Conceptually:

```text
                 my-app:1.0
                 Image layers
                /     |     \
               /      |      \
              v       v       v
            app1     app2     app3
             |        |        |
           RW-1     RW-2     RW-3
```

The containers can share the image's underlying read-only data while having independent writable layers.

This is a major reason Docker can efficiently create many containers from the same image.

---

# 18. What about logs?

Suppose your application writes:

```text
/app/logs/application.log
```

If `/app/logs` is just part of the container filesystem:

```text
Container
   |
   +-- /app/logs/application.log
```

then the log is stored in the container's writable storage.

If the container is removed, that data can disappear.

In production, we generally don't want important application data to depend on the lifetime of a container.

That's why production architectures usually externalize important state:

```text
Container
   |
   +-- application
   |
   +-- logs → logging system
   |
   +-- database → persistent storage
```

We'll eventually connect this to:

* Docker volumes
* log drivers
* centralized logging
* Kubernetes persistent volumes

But those are later topics.

---

# 19. What about `/proc`, `/sys`, and `/dev`?

You may notice something interesting when exploring a container:

```bash
ls /
```

and see:

```text
proc
sys
dev
```

These aren't simply ordinary application directories containing regular files.

Linux uses special virtual/pseudo filesystems for things such as:

```text
/proc
/sys
/dev
```

Containers use Linux kernel mechanisms such as namespaces to provide an isolated view of many of these resources.

You don't need to understand the kernel implementation yet.

At the beginner level, remember:

> **A container gets its own filesystem view, but it still uses the host's Linux kernel.**

This is an important distinction.

---

# 20. Container filesystem does NOT mean a complete VM

This is another important misconception.

A virtual machine might look like:

```text
VM
├── Guest OS
├── Guest kernel
├── Guest filesystem
└── Application
```

A container is different:

```text
Container
├── Container filesystem
├── Application
└── Uses host kernel
```

So a container has its own filesystem environment, but it does **not** normally contain its own separate Linux kernel.

This is one reason containers are generally much lighter than VMs.

---

# 21. What Docker is actually doing

Let's simplify the whole process.

When you run:

```bash
docker run my-app:1.0
```

conceptually Docker/container runtime does something like:

```text
1. Find the image
       ↓
2. Prepare image layers
       ↓
3. Add a writable container layer
       ↓
4. Create the container's isolated environment
       ↓
5. Start the configured process
```

The application then sees something like:

```text
/
├── etc
├── usr
├── var
├── tmp
└── app
```

while underneath:

```text
Container filesystem
        |
        +-- writable container layer
        |
        +-- read-only image layers
```

That is the mental model we want.

---

# 22. Hands-on experiment

This is a very good experiment to perform on your Docker lab.

Start a container:

```bash
docker run -it --name filesystem-test ubuntu:24.04 bash
```

Inside:

```bash
echo "hello from container" > /hello.txt
```

Check:

```bash
cat /hello.txt
```

You should get:

```text
hello from container
```

Exit:

```bash
exit
```

Now start the same container again:

```bash
docker start -ai filesystem-test
```

Check:

```bash
cat /hello.txt
```

It should still exist.

Why?

Because you stopped the container, but did **not delete it**.

Its writable layer still exists.

Now exit:

```bash
exit
```

Remove the container:

```bash
docker rm filesystem-test
```

Create a new container from the same image:

```bash
docker run -it --name filesystem-test2 ubuntu:24.04 bash
```

Check:

```bash
ls /hello.txt
```

It should not exist.

Why?

```text
Container 1
    |
    +-- writable layer
          |
          +-- /hello.txt
```

Container 1 was deleted.

The image:

```text
ubuntu:24.04
```

was never changed.

The new container gets a new writable layer.

---

# 23. The lifecycle picture

This experiment gives us a very important model:

```text
                 Image
                  │
             docker run
                  │
                  v
            +-----------+
            | Container |
            |           |
            | RW Layer  |
            +-----------+
                  │
             docker stop
                  │
                  v
            Container exists
            RW layer remains
                  │
             docker start
                  │
                  v
            Same filesystem
                  │
              docker rm
                  │
                  v
            RW layer removed
```

So:

```text
STOP ≠ DELETE
```

Stopping a container does not normally destroy its writable filesystem.

Deleting the container does.

---

# 24. The connection to CI/CD

Now let's connect this to CI/CD.

Suppose Jenkins deploys:

```text
my-app:42
```

Kubernetes creates a container from that image.

The container gets:

```text
Image
  +
Writable layer
  =
Container filesystem
```

Now imagine your application writes important data directly into:

```text
/app/data
```

and the deployment replaces the container.

The old container disappears.

Its writable layer disappears.

Therefore, application state stored there may disappear.

This is one reason modern CI/CD systems favor **stateless application containers**.

A common architecture is:

```text
Container
   |
   +-- application code
   |
   +-- temporary files
   |
   +-- ephemeral data
```

while persistent state lives elsewhere:

```text
Database
Object Storage
Persistent Volume
External Logging System
```

This becomes extremely important when we reach Kubernetes.

---

# 25. Beginner concepts vs advanced concepts

### You should understand now

Make sure these are clear:

1. A container gets a filesystem based on its image.
2. Image layers are read-only.
3. A running container gets a writable layer.
4. Changes made inside the container go into its writable layer.
5. Different containers have different writable layers.
6. Deleting the container removes its writable layer.
7. The image itself isn't modified when a container changes files.
8. A container's filesystem is not automatically the host filesystem.
9. Important persistent data should not depend on the container writable layer.
10. Containers use the host's Linux kernel rather than having their own separate kernel.

### Advanced — leave for later

We don't need to dive into these yet:

* overlay2
* OverlayFS
* lowerdir
* upperdir
* merged directory
* copy-up behavior
* whiteout files
* inode behavior
* storage drivers
* containerd snapshotters
* filesystem performance characteristics
* rootless storage
* SELinux labeling

Those are useful later, especially for production troubleshooting, but they would distract from the fundamental mental model right now.

---

# 26. The complete Phase 1 mental model so far

We can now connect **all six topics**:

```text
                 Dockerfile
                     |
                     v
               Build Context
                     |
                     v
                Docker Build
                     |
                     v
                Image Layers
                     |
                     v
                 Docker Image
                     |
                     | push
                     v
              Image Registry
                     |
                     | pull
                     v
                 Docker Image
                     |
                  docker run
                     |
                     v
          +----------------------+
          | Container Filesystem |
          |                      |
          | Writable Layer       |
          |        +             |
          | Image Read-Only      |
          | Layers               |
          +----------------------+
                     |
                     v
                Application
```

This is the important progression:

> **Dockerfile tells Docker how to build → build context supplies the input → Docker creates image layers → those layers form an image → registry distributes the image → container gets a filesystem from that image plus a writable layer.**

---

# 27. The one sentence to remember

If you remember only one thing from Step 6:

> **A container gets a filesystem from the image's read-only layers plus its own writable layer, and that writable layer belongs to the lifetime of the container.**

And the most important diagram is:

```text
             CONTAINER FILESYSTEM

        ┌─────────────────────────┐
        │   Writable Layer        │  ← container changes
        ├─────────────────────────┤
        │   Image Layer 4         │  ← read-only
        ├─────────────────────────┤
        │   Image Layer 3         │  ← read-only
        ├─────────────────────────┤
        │   Image Layer 2         │  ← read-only
        ├─────────────────────────┤
        │   Image Layer 1         │  ← read-only
        └─────────────────────────┘
```

The next topic, **Step 7 — Base Images**, will answer an important question that naturally follows from this:

> **Where do those initial image filesystem layers actually come from?**

That will take us into `FROM ubuntu`, `FROM alpine`, `FROM python`, and what a **base image** really means.
