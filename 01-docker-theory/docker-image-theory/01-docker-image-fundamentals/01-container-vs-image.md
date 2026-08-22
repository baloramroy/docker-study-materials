# Phase 1 — Part 1: Container vs Image

We will stay **strictly in roadmap order** and will not go deeply into Dockerfile, layers, filesystem, registry, or other later concepts yet.

```text
Phase 1 — Docker Fundamentals

1.  Container vs Image       ← NOW
2.  Dockerfile
3.  Build context
4.  Docker image layers
5.  Docker image registry
6.  Container filesystem
7.  Base images
8.  Image tags
```

The goal of this topic is to build one foundational mental model:

> **What is a Docker image, what is a Docker container, and how are they related?**

---

# 1. Start With the Simplest Mental Model

The most important relationship is:

```text
Docker Image
     │
     │ used to create
     ▼
Docker Container
```

Or even shorter:

```text
Image → Container
```

Think of it this way:

> **Image = packaged artifact**
> **Container = running instance of that artifact**

This distinction will appear throughout everything we learn later.

---

# 2. What Is a Docker Image?

A **Docker image** is a packaged artifact that contains the filesystem content and configuration needed to create a container.

For example, imagine an application:

```text
my-app/
├── app.py
└── requirements.txt
```

Eventually, we can build an image containing the application's required environment.

Conceptually:

```text
┌──────────────────────────────┐
│        Docker Image          │
│                              │
│  Application files           │
│  Runtime                     │
│  Dependencies                │
│  Filesystem content          │
│  Configuration / metadata    │
│                              │
└──────────────────────────────┘
```

The important thing is:

> **The image is the packaged artifact.**

It is not the running application.

---

# 3. What Is a Container?

A **container** is an isolated runtime environment created from a Docker image.

For example:

```text
              Docker Image
                   │
          ┌────────┼────────┐
          │        │        │
          ▼        ▼        ▼
      Container Container Container
         #1         #2         #3
```

All three containers can be created from the same image.

For example:

```text
my-app:1.0
    │
    ├── container-1
    ├── container-2
    └── container-3
```

The image provides the packaged starting point.

Each container is a separate runtime instance.

---

# 4. Image vs Container — The Core Difference

Let's make the distinction very explicit.

| Docker Image                 | Docker Container                   |
| ---------------------------- | ---------------------------------- |
| Packaged artifact            | Runtime instance                   |
| Used to create containers    | Created from an image              |
| Usually treated as immutable | Has a writable runtime layer       |
| Can be stored/distributed    | Can be started/stopped/removed     |
| Can create many containers   | Represents one particular instance |

The most important relationship is:

```text
                    IMAGE
                      │
          ┌───────────┼───────────┐
          │           │           │
          ▼           ▼           ▼
      Container   Container   Container
          #1          #2          #3
```

One image can therefore be used to create many containers.

---

# 5. Why Does Docker Separate Images and Containers?

This is one of the most important design ideas in Docker.

Imagine you build:

```text
my-app:1.0
```

You now have a packaged application artifact.

You could use that same artifact for:

```text
Testing
Staging
Production
```

Conceptually:

```text
                  my-app:1.0
                      │
          ┌───────────┼───────────┐
          │           │           │
          ▼           ▼           ▼
       Testing     Staging    Production
```

Each environment can create a container from the same image.

This gives us an important CI/CD principle:

> **Build the artifact once, then promote that same artifact through environments.**

The image is the artifact.

The container is the runtime instance.

---

# 6. Image Is Not a Running Application

This is a common beginner confusion.

Suppose you have:

```text
my-app:1.0
```

That does **not** necessarily mean:

```text
"my application is currently running."
```

It means:

```text
"I have an image from which a container can be created."
```

The basic workflow is:

```text
Image
  │
  │ docker run
  ▼
Container
  │
  ▼
Application process
```

So:

```bash
docker build
```

and:

```bash
docker run
```

have different jobs.

---

# 7. What Does `docker build` Do?

When you eventually run:

```bash
docker build -t my-app:1.0 .
```

the result is an **image**.

Conceptually:

```text
Source + Dockerfile
        │
        │ docker build
        ▼
   Docker Image
   my-app:1.0
```

The command does not directly mean:

```text
"start my application."
```

It means:

> **Build the image.**

We will study exactly how that build happens when we reach Dockerfile, build context, and image layers.

---

# 8. What Does `docker run` Do?

Once the image exists:

```text
my-app:1.0
```

you can run:

```bash
docker run my-app:1.0
```

Conceptually:

```text
Docker Image
     │
     │ docker run
     ▼
Docker Container
     │
     ▼
Application process
```

So the basic lifecycle is:

```text
docker build
      │
      ▼
    Image
      │
      │ docker run
      ▼
  Container
```

This is one of the most important command relationships in Docker.

---

# 9. A Concrete Example

Suppose you have:

```text
my-app/
└── app.py
```

Eventually, you create the Docker build definition and execute:

```bash
docker build -t my-app:1.0 .
```

You now have:

```text
my-app:1.0
```

which is an image.

Then:

```bash
docker run my-app:1.0
```

creates a container from that image.

The complete simplified flow is:

```text
             Application
             source code
                  │
                  ▼
              Dockerfile
                  │
                  │ docker build
                  ▼
        ┌──────────────────┐
        │ Docker Image      │
        │ my-app:1.0        │
        └──────────────────┘
                  │
                  │ docker run
                  ▼
        ┌──────────────────┐
        │ Docker Container  │
        │ running app       │
        └──────────────────┘
```

---

# 10. One Image Can Create Multiple Containers

This is extremely important.

Suppose we have:

```text
my-app:1.0
```

We can create:

```text
my-app:1.0
    │
    ├── Container A
    ├── Container B
    └── Container C
```

For example:

```bash
docker run --name app-1 my-app:1.0
docker run --name app-2 my-app:1.0
docker run --name app-3 my-app:1.0
```

The three containers originate from the same image.

Conceptually:

```text
                 my-app:1.0
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       app-1       app-2       app-3
```

This is one of the reasons images are useful as reusable application artifacts.

---

# 11. Are Those Containers the Same?

No.

They came from the same image, but they are **different container instances**.

Think:

```text
Same Image
    │
    ├── Container A
    │
    ├── Container B
    │
    └── Container C
```

Each container has its own:

* container identity
* runtime state
* writable changes
* process state
* network identity/configuration

So:

> **Same image does not mean same container.**

---

# 12. A Useful Analogy — Blueprint vs Building

A better analogy than "ISO" for the initial mental model is:

```text
Blueprint
    │
    ├── Building A
    ├── Building B
    └── Building C
```

Similarly:

```text
Docker Image
    │
    ├── Container A
    ├── Container B
    └── Container C
```

The image provides the common packaged starting point.

Each container is an individual runtime instance.

This analogy isn't technically perfect, but it helps establish the relationship.

---

# 13. Don't Think of a Container as a Virtual Machine

Another important beginner misconception:

> **A Docker container is not simply a lightweight virtual machine.**

A virtual machine generally contains:

```text
VM
├── Application
├── Libraries
├── Guest OS
└── Virtual hardware
```

A container uses the host kernel through operating-system isolation mechanisms.

Very simplified:

```text
Virtual Machines

Application
    ↓
Guest OS
    ↓
Hypervisor
    ↓
Host


Containers

Application
    ↓
Container
    ↓
Host Kernel
```

We don't need to go deeper into namespaces, cgroups, container runtimes, or kernel behavior yet.

Those are advanced topics.

For now, remember:

> **Container ≠ VM.**

---

# 14. Is the Image a Complete Operating System?

This is another common misconception.

You may hear:

> "This image contains Ubuntu."

For example:

```text
ubuntu:24.04
```

But that doesn't mean the image contains a complete bootable Ubuntu virtual machine.

A container image provides a filesystem/user-space environment and configuration for containers.

The container does not boot its own kernel like a VM.

So don't mentally model:

```text
Docker Image
    =
Complete VM
```

Instead:

```text
Docker Image
    =
Packaged filesystem + configuration
for creating a container
```

---

# 15. What Happens When a Container Starts?

At a high level:

```text
Docker Image
      │
      │ create/start
      ▼
Container
      │
      ▼
Main process starts
```

For example, an image might specify that the application should start with:

```text
python app.py
```

When a container is started, that process runs inside the container environment.

The important relationship is:

```text
Image
  │
  │ provides filesystem/configuration
  ▼
Container
  │
  │ provides runtime environment
  ▼
Application process
```

We'll study the exact filesystem and startup behavior later.

---

# 16. The Container Has Its Own Runtime Changes

This is an important difference between the image and the container.

Suppose an image contains:

```text
/app/app.py
```

When a container is created, Docker provides a writable layer on top of the image's read-only filesystem layers.

Conceptually:

```text
Container
┌───────────────────────────┐
│ Writable container layer  │
├───────────────────────────┤
│ Image filesystem          │
│                           │
│ Application               │
│ Dependencies              │
│ Runtime files             │
└───────────────────────────┘
```

If the application modifies a file inside the container, that change belongs to the container's writable layer rather than changing the original image.

This is an important concept.

**But don't go too deep into it yet.**

We'll study container filesystem behavior in Step 6 and image layers in Step 4.

---

# 17. Image vs Container: Immutability Mental Model

A useful beginner mental model is:

```text
Image
    ↓
Read-only packaged artifact

Container
    ↓
Runtime instance with writable changes
```

So if you have:

```text
my-app:1.0
```

and create:

```text
container-A
```

then modify something inside `container-A`, you have **not modified the original image**.

You modified that container's runtime state.

This is one reason the image can safely be reused to create additional containers.

---

# 18. What Happens If the Container Is Deleted?

Suppose:

```text
my-app:1.0
      │
      ▼
container-A
```

You make changes inside the container.

Then:

```bash
docker rm container-A
```

The container is removed.

The image:

```text
my-app:1.0
```

can still exist.

So:

```text
Image
 │
 ├── Container A  ← deleted
 │
 └── Container B  ← can still be created
```

The image and container have different lifecycles.

This distinction becomes very important in real Docker usage.

---

# 19. Container Lifecycle vs Image Lifecycle

Think of the image as a reusable artifact:

```text
Build
  │
  ▼
Image
  │
  ├── create container
  ├── create container
  └── create container
```

Containers have their own lifecycle:

```text
Created
   │
   ▼
Running
   │
   ▼
Stopped
   │
   ▼
Removed
```

The image doesn't disappear just because one container stops or is removed.

---

# 20. Image Distribution

Images are also designed to be distributed.

For example:

```text
Developer
    │
    │ build
    ▼
Docker Image
    │
    │ push
    ▼
Container Registry
    │
    │ pull
    ▼
Production Server
    │
    │ run
    ▼
Container
```

This is the foundation of the CI/CD workflow we'll eventually build.

For now, just understand:

> **The image is the artifact that can be built, stored, transferred, and used to create containers.**

We'll study registries separately in Step 5.

---

# 21. The CI/CD Mental Model

This distinction becomes especially important in CI/CD.

A simplified pipeline is:

```text
Developer
    │
    │ git push
    ▼
CI Pipeline
    │
    ├── Test
    │
    └── docker build
              │
              ▼
         Docker Image
              │
              ├── Scan
              ├── Tag
              └── Push
                     │
                     ▼
               Image Registry
                     │
                     ▼
                Deployment
                     │
                     ▼
                 Container
```

Notice the separation:

```text
CI
 │
 └── builds IMAGE

CD
 │
 └── deploys IMAGE as CONTAINER
```

That is a very useful mental model for your CI/CD learning path.

---

# 22. Why "Build Once, Deploy Many" Matters

Suppose CI builds:

```text
my-app:1.0
```

and tests that exact image.

Then staging uses:

```text
my-app:1.0
```

and production also uses:

```text
my-app:1.0
```

Conceptually:

```text
                  my-app:1.0
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       Testing      Staging     Production
```

You're not rebuilding the application separately for every environment.

You're promoting the same artifact.

This helps reduce the risk of:

```text
Tested artifact ≠ Production artifact
```

Docker images are very useful for this model.

---

# 23. Beginner Misconception: "Dockerfile = Image"

No.

They are three different things:

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

### Dockerfile

Instructions describing how an image should be built.

### Image

The built artifact.

### Container

An instance created from the image.

We'll study Dockerfile next.

---

# 24. Beginner Misconception: "Container = Image"

No.

An image is the packaged artifact.

A container is an instance created from that image.

```text
Image
  │
  ├── Container A
  ├── Container B
  └── Container C
```

One image can produce many containers.

---

# 25. Beginner Misconception: "Running a Container Changes the Image"

Normally, no.

Suppose:

```text
my-app:1.0
```

creates:

```text
container-A
```

and the application changes a file.

That change belongs to the container's writable layer.

The original image remains unchanged.

Conceptually:

```text
             my-app:1.0
             Docker Image
                  │
                  ▼
             container-A
                  │
                  │ writes file
                  ▼
          Container writable layer
```

This distinction will become much clearer when we study image layers and the container filesystem.

---

# 26. Beginner Misconception: "Stopping a Container Deletes the Image"

No.

For example:

```bash
docker stop my-container
```

stops the container.

It does not mean:

```text
Image deleted
```

You could start the container again.

Likewise:

```bash
docker rm my-container
```

removes that container, but the image can still exist.

So:

```text
Image lifecycle
      ≠
Container lifecycle
```

---

# 27. Beginner Misconception: "One Image = One Container"

No.

This is one of the most important things to understand.

```text
                 Image
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
    Container   Container   Container
```

This is completely normal.

In production, you may have multiple containers running from the same image.

---

# 28. Beginner Misconception: "Container Is the Application"

A container provides the environment in which the application's main process runs.

More precisely:

```text
Image
  │
  ▼
Container
  │
  ▼
Application process
```

The container isn't simply the application binary itself.

It's the isolated runtime environment around the process.

---

# 29. What Docker Is Actually Doing

Let's build the simplified lifecycle carefully.

### Step 1 — Build

You have source code and Docker build instructions:

```text
Source code
     +
Dockerfile
     │
     │ docker build
     ▼
Docker Image
```

### Step 2 — Create a container

```bash
docker create my-app:1.0
```

Conceptually:

```text
Image
  │
  ▼
Container created
```

### Step 3 — Start it

```bash
docker start <container>
```

Conceptually:

```text
Container
    │
    ▼
Main process starts
```

### Or combine create + start

```bash
docker run my-app:1.0
```

Conceptually:

```text
docker run
    │
    ├── create container
    │
    └── start container
```

So `docker run` is not the same thing as `docker build`.

---

# 30. The Most Important Command Relationship

Keep this in your head:

```text
docker build
      │
      ▼
    IMAGE
      │
      │ docker run
      ▼
  CONTAINER
```

Or:

```text
BUILD → IMAGE → RUN → CONTAINER
```

This is the foundation for the entire Docker image-building topic.

---

# 31. A More Complete Mental Model

Now combine everything we've learned:

```text
                    Source Code
                         │
                         │
                         ▼
                    Dockerfile
                         │
                         │ docker build
                         ▼
                ┌─────────────────┐
                │   Docker Image  │
                │                 │
                │  Packaged       │
                │  artifact       │
                └─────────────────┘
                         │
                  ┌──────┼──────┐
                  │      │      │
                  ▼      ▼      ▼
               Container Container Container
                  │      │      │
                  ▼      ▼      ▼
               Process  Process  Process
```

Don't worry yet about exactly how the Dockerfile becomes the image.

That is what the next topics will explain.

---

# 32. Where the Next Topics Fit

Now we can see why the roadmap is ordered this way.

### Step 1 — Container vs Image

```text
What are the two things?
```

### Step 2 — Dockerfile

```text
How do we describe how to build an image?
```

### Step 3 — Build Context

```text
What files are available to the build?
```

### Step 4 — Image Layers

```text
How is the image actually constructed?
```

### Step 5 — Image Registry

```text
Where can the image be stored and distributed?
```

### Step 6 — Container Filesystem

```text
What does the filesystem look like when a container runs?
```

### Step 7 — Base Images

```text
Where does an image start from?
```

### Step 8 — Image Tags

```text
How do we identify/version images?
```

The progression is deliberate.

---

# 33. What We Are NOT Learning Yet

To keep this topic clean, we're **not going deeply into**:

* Dockerfile instructions
* build context
* image layers
* layer caching
* registries
* container filesystem internals
* namespaces
* cgroups
* containerd
* OCI image specifications
* overlay filesystems
* BuildKit internals
* image digests

You'll encounter many of these later.

For now, the objective is simply:

> **Understand the difference between an image and a container.**

---

# 34. Beginner Level vs Advanced Level

### You should understand now

You should be comfortable with:

* What a Docker image is
* What a Docker container is
* Image vs container
* One image → many containers
* `docker build` → image
* `docker run` → container
* Image as a deployable artifact
* Container as a runtime instance
* Why the separation matters in CI/CD
* Why container changes don't normally modify the original image

### Advanced — leave for later

Don't worry yet about:

* namespaces
* cgroups
* overlay2
* OCI specifications
* containerd internals
* runc
* content-addressable storage
* image manifests
* image digests
* snapshotters

Those concepts are useful eventually, but introducing them now would obscure the fundamental model.

---

# 35. Hands-On: See the Relationship Yourself

If Docker is installed, start by checking your images:

```bash
docker images
```

You can then run:

```bash
docker run --name demo-container nginx
```

Conceptually:

```text
nginx image
     │
     │ docker run
     ▼
demo-container
```

Check containers:

```bash
docker ps
```

Then stop it:

```bash
docker stop demo-container
```

Check all containers:

```bash
docker ps -a
```

The container is now stopped, but the image can still exist.

You can remove the container:

```bash
docker rm demo-container
```

The image can still remain available.

This simple exercise demonstrates the different lifecycles of:

```text
Image
  vs
Container
```

---

# 36. One Small Experiment: Same Image, Two Containers

Run:

```bash
docker run -d --name app-1 nginx
```

and:

```bash
docker run -d --name app-2 nginx
```

Now:

```bash
docker ps
```

You should have two containers:

```text
nginx image
    │
    ├── app-1
    └── app-2
```

The important observation is:

> **You did not need two different images to create two containers.**

Both came from the same image.

---

# 37. The Three Terms You Must Remember

At this stage, these three definitions should be very clear.

### Dockerfile

```text
Instructions for building an image
```

### Image

```text
Packaged artifact used to create containers
```

### Container

```text
Runtime instance created from an image
```

The relationship:

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

This is the foundation for the rest of Phase 1.

---

# 38. The One-Sentence Takeaway

If you remember only one thing from Step 1:

> **A Docker image is the packaged artifact, while a container is a runtime instance created from that image.**

Or:

```text
BUILD AN IMAGE.
RUN A CONTAINER.
```

And the complete mental model is:

```text
                    Dockerfile
                         │
                         │ docker build
                         ▼
                ┌─────────────────┐
                │   Docker Image  │
                │                 │
                │ "What we build" │
                └─────────────────┘
                         │
                         │ docker run
                         ▼
                ┌─────────────────┐
                │    Container    │
                │                 │
                │ "What we run"   │
                └─────────────────┘
```

---

# Phase 1 — Progress

We're following the exact order from your guide:

```text
1. Container vs Image       ✅ CURRENT
2. Dockerfile               ⏳
3. Build context            ⏳
4. Docker image layers      ⏳
5. Docker image registry    ⏳
6. Container filesystem     ⏳
7. Base images              ⏳
8. Image tags               ⏳
```

Before moving to **2. Dockerfile**, make sure this distinction is solid:

```text
             SOURCE CODE
                  │
                  ▼
             Dockerfile
                  │
                  │ docker build
                  ▼
            ┌───────────┐
            │   IMAGE   │
            └───────────┘
                  │
                  │ docker run
                  ▼
            ┌───────────┐
            │ CONTAINER │
            └───────────┘
```

**Next in the strict roadmap: Dockerfile.**
