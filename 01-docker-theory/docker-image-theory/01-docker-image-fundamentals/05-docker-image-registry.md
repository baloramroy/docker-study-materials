# Phase 1 — Part 5: Docker Image Registry

## Where we are

```text
Phase 1 — Docker Fundamentals

01. Container vs Image       ✅
02. Dockerfile               ✅
03. Build Context            ✅
04. Docker Image Layers      ✅
05. Docker Image Registry    ← NOW
06. Container Filesystem
07. Base Images
08. Image Tags
```

This topic is particularly important for CI/CD because building an image is only one part of the process.

A typical workflow is:

```text
Source Code
    ↓
Docker Build
    ↓
Docker Image
    ↓
Push to Registry
    ↓
Registry
    ↓
Pull Image
    ↓
Deployment
    ↓
Container
```

---

# 1. What Is a Docker Image Registry?

A **Docker image registry is a service that stores and distributes container images.**

You can think of it as a central location where Docker images are:

* pushed
* stored
* pulled
* shared

For example:

```text
Developer / CI Server
        │
        │ docker push
        ▼
   Image Registry
        │
        │ docker pull
        ▼
Server / Kubernetes
```

The registry solves an important problem:

> **How does the machine that builds an image make that image available to the machine that will run it?**

---

# 2. Why Do We Need a Registry?

Suppose your CI server builds:

```text
my-app:1.0
```

The image exists on the CI server:

```text
CI Server
└── my-app:1.0
```

But your production server doesn't automatically have it:

```text
Production Server
└── ??? 
```

The CI server could send the image directly somehow, but that's not the standard approach.

Instead:

```text
                    docker build
                         │
                         ▼
                    CI Server
                    my-app:1.0
                         │
                    docker push
                         │
                         ▼
                  Image Registry
                         │
                    docker pull
                         │
                         ▼
                 Production Server
                    my-app:1.0
```

The registry acts as the **central distribution point**.

---

# 3. Registry vs Repository vs Image

These three terms are often confused.

Suppose you have:

```text
docker.io/mycompany/my-app:1.0
```

Conceptually:

```text
docker.io
    │
    └── Registry

mycompany/my-app
    │
    └── Repository

1.0
    │
    └── Tag
```

So:

### Registry

Where images are stored and distributed.

### Repository

A named collection/location for related images.

### Image

A particular image artifact stored in the repository.

### Tag

A human-readable reference associated with an image.

We'll study tags properly in **Part 8**.

---

# 4. Docker Hub

One of the most commonly used public registries is Docker's Docker Hub.

For example:

```text
docker.io/library/nginx
```

is the repository for the official Nginx image.

When you previously ran:

```bash
docker pull nginx
```

Docker was effectively retrieving an image from a registry.

Conceptually:

```text
docker pull nginx
       │
       ▼
Docker Hub
       │
       ▼
nginx image
       │
       ▼
Local Docker host
```

You don't always have to write `docker.io` explicitly because Docker uses its default registry when appropriate.

---

# 5. Public vs Private Registries

Registries can be broadly divided into:

```text
Public Registry
       │
       └── publicly available images

Private Registry
       │
       └── organization-controlled images
```

### Public registry

Examples include:

* Docker Hub
* GitHub Container Registry
* public cloud registries

### Private registry

Organizations commonly use private registries to store their internal application images.

Examples include:

* Harbor
* Amazon ECR
* Azure Container Registry
* Google Artifact Registry
* private Docker Registry

For an enterprise CI/CD environment, a private registry is extremely common.

---

# 6. Why Would a Company Use a Private Registry?

Imagine your company builds:

```text
payment-service
customer-service
notification-service
```

You don't want those internal images publicly accessible.

Instead:

```text
                    CI Pipeline
                        │
                   docker build
                        │
                        ▼
                  Private Registry
                  ┌───────────────┐
                  │ payment       │
                  │ customer      │
                  │ notification  │
                  └───────────────┘
                        │
                        ▼
                 Production
```

The registry can also provide:

* authentication
* access control
* image retention
* vulnerability scanning
* audit information
* repository management

The exact features depend on the registry.

---

# 7. Image Registry in CI/CD

This is the part most relevant to your goal.

A typical pipeline looks like:

```text
Developer
    │
    │ git push
    ▼
Git Repository
    │
    ▼
CI Server
    │
    │ docker build
    ▼
Docker Image
    │
    │ docker push
    ▼
Image Registry
    │
    │ docker pull
    ▼
Deployment Environment
```

For example:

```text
Jenkins
   │
   │ Build
   ▼
my-app:build-105
   │
   │ Push
   ▼
Harbor / ECR / Docker Hub
   │
   │ Pull
   ▼
Kubernetes
```

This is the basic **build → store → deploy** model you'll encounter repeatedly in CI/CD.

---

# 8. Push an Image to a Registry

Let's use Docker Hub as a learning example.

Suppose you already have:

```bash
docker images
```

and:

```text
REPOSITORY   TAG
my-nginx     1.0
```

A registry expects an image reference containing the appropriate repository path.

Conceptually:

```text
registry/repository:tag
```

For example:

```text
docker.io/myusername/my-nginx:1.0
```

You can tag the local image with that registry name:

```bash
docker tag my-nginx:1.0 docker.io/myusername/my-nginx:1.0
```

Then authenticate:

```bash
docker login
```

And push:

```bash
docker push docker.io/myusername/my-nginx:1.0
```

The flow is:

```text
Local Image
my-nginx:1.0
     │
     │ docker tag
     ▼
docker.io/myusername/my-nginx:1.0
     │
     │ docker push
     ▼
Docker Registry
```

**Important:** `docker tag` does not rebuild or duplicate the image contents. It gives an image another reference/name.

We'll revisit this distinction when we study tags.

---

# 9. Pull an Image From a Registry

Another machine can retrieve the image:

```bash
docker pull docker.io/myusername/my-nginx:1.0
```

Conceptually:

```text
Registry
    │
    │ docker pull
    ▼
Docker Host
    │
    ▼
my-nginx:1.0
```

Then:

```bash
docker run myusername/my-nginx:1.0
```

can create a container from it.

So the complete flow is:

```text
Build
  ↓
Image
  ↓
Push
  ↓
Registry
  ↓
Pull
  ↓
Image
  ↓
Run
  ↓
Container
```

---

# 10. Registry Does Not Run Your Container

This is an important distinction.

A registry primarily stores and distributes images.

It does **not** normally mean:

```text
Registry → runs your application
```

Instead:

```text
Registry
   │
   │ stores/distributes
   ▼
Docker Image
   │
   │ pulled by
   ▼
Docker Host / Kubernetes
   │
   │ runs
   ▼
Container
```

So:

* **Registry** → stores/distributes images
* **Docker Engine** → runs containers
* **Kubernetes** → orchestrates containers/workloads

We'll eventually connect these concepts in your CI/CD and Kubernetes learning.

---

# 11. What Actually Gets Pushed?

Remember Part 4.

An image consists of layers.

When you push an image:

```bash
docker push myusername/my-nginx:1.0
```

Docker pushes the image's required content to the registry.

Conceptually:

```text
Docker Image
┌───────────────────┐
│ Layer 3           │
│ Layer 2           │
│ Layer 1           │
└───────────────────┘
        │
        │ push
        ▼
   Image Registry
```

Because layers can be shared, registries can avoid storing duplicate layer content unnecessarily.

For example:

```text
Image A ──┐
          ├── shared base layer
Image B ──┘
```

The registry can store that shared content efficiently.

This is one reason understanding **layers before registries** was important.

---

# 12. Image Digest

There's one more concept we need to introduce: **digest**.

An image can be referenced by a tag:

```text
my-app:1.0
```

but it can also be identified by a content digest:

```text
my-app@sha256:abc123...
```

The digest is based on the image's content/manifest and provides a content-addressed identity.

Conceptually:

```text
Tag:
my-app:1.0
     │
     └── human-friendly reference

Digest:
my-app@sha256:...
     │
     └── content-based reference
```

This distinction is extremely important in production deployments because tags can be moved to point to different image versions, while a digest identifies a specific image manifest.

We will go deeper into this in **Part 8 — Image Tags**, but for now just remember:

> **Tag = convenient reference. Digest = immutable content-based reference.**

---

# 13. Why Tags and Digests Matter in CI/CD

Suppose CI builds:

```text
my-app:1.0
```

and pushes it.

Later, someone pushes another image using the same tag:

```text
my-app:1.0
```

The tag can now refer to a different image.

Therefore:

```text
Tag
 │
 └── can move

Digest
 │
 └── identifies specific content
```

For production deployments, digest-based references can provide stronger reproducibility.

We won't go deeper here because **Part 8 is specifically dedicated to image tags**.

---

# 14. Hands-on Exercise — Local Registry Flow

Before using a public registry, let's understand the workflow using your local Docker environment.

First build:

```bash
docker build -t registry-demo:1.0 .
```

Check:

```bash
docker images
```

You now have:

```text
registry-demo:1.0
```

The image is local:

```text
Local Docker Host
└── registry-demo:1.0
```

The important next step would normally be:

```text
docker tag
      ↓
docker push
      ↓
Registry
```

For the moment, don't push anything publicly unless you want to.

The key workflow is what matters:

```text
Local Build
    ↓
Local Image
    ↓
Tag for Registry
    ↓
Push
    ↓
Registry
```

---

# 15. Hands-on Exercise — Understand the Image Reference

Run:

```bash
docker images
```

You might have:

```text
REPOSITORY    TAG
registry-demo 1.0
```

Now:

```bash
docker tag registry-demo:1.0 myregistry.example.com/myteam/registry-demo:1.0
```

Run:

```bash
docker images
```

You may now see both:

```text
REPOSITORY                                  TAG
registry-demo                              1.0
myregistry.example.com/myteam/registry-demo 1.0
```

This does **not** mean Docker created a second independent image.

Both references can point to the same underlying image content.

This is an important point to remember before Part 8.

---

# 16. The Standard CI/CD Pattern

Now combine everything we've learned:

```text
                   Git Repository
                         │
                         │ source code
                         ▼
                    CI Pipeline
                         │
                         │ docker build
                         ▼
                    Docker Image
                         │
                         │ docker push
                         ▼
                  Image Registry
                         │
                         │ docker pull
                         ▼
                Deployment Environment
                         │
                         │ run
                         ▼
                     Container
```

This is one of the fundamental patterns you'll see in modern CI/CD.

For example:

```text
Jenkins
   │
   ├── checkout
   ├── test
   ├── docker build
   └── docker push
             │
             ▼
          Harbor
             │
             ▼
        Kubernetes
```

Later, when you build your CI/CD lab, this exact concept will become practical.

---

# 17. Common Mistakes

### Mistake 1 — Thinking the registry builds the image

Incorrect:

```text
Registry → builds image
```

Normally:

```text
CI / Developer
      │
      │ docker build
      ▼
    Image
      │
      │ docker push
      ▼
  Registry
```

---

### Mistake 2 — Thinking Docker Hub is Docker itself

Docker is the technology/platform.

Docker Hub is one registry service.

Conceptually:

```text
Docker
  │
  └── Docker Hub
       └── Registry service
```

There are many other registries.

---

### Mistake 3 — Thinking `docker push` sends the Dockerfile

It doesn't.

The normal flow is:

```text
Dockerfile
    ↓
docker build
    ↓
Image
    ↓
docker push
    ↓
Registry
```

The registry primarily receives the image and its metadata/content, not your Dockerfile as the build instructions.

---

### Mistake 4 — Thinking a registry is the same as a repository

They're different:

```text
Registry
   │
   ├── Repository A
   │
   ├── Repository B
   │
   └── Repository C
```

A registry hosts repositories.

---

# 18. What You Should Know Now

You should now understand:

* A **registry stores and distributes Docker images**.
* A **repository** is a named location/collection for related images within a registry.
* `docker push` uploads an image to a registry.
* `docker pull` retrieves an image from a registry.
* A registry is central to the CI/CD build → deploy workflow.
* Public and private registries serve different organizational needs.
* Image layers are part of what gets distributed.
* A tag is a convenient image reference.
* A digest identifies specific image content.
* A registry does not normally run your containers.

The key flow is:

```text
Dockerfile
    ↓
docker build
    ↓
Docker Image
    ↓
docker push
    ↓
Registry
    ↓
docker pull
    ↓
Docker Image
    ↓
docker run
    ↓
Container
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
06. Container Filesystem     ← NEXT
07. Base Images
08. Image Tags
```

We've now completed the part of the roadmap that explains **how an image goes from source code to a shareable artifact**:

```text
Dockerfile
     ↓
Build Context
     ↓
Image Layers
     ↓
Docker Image
     ↓
Registry
```

**Next: Part 6 — Container Filesystem.**

There we'll examine what actually happens to the filesystem when you run an image as a container, including the relationship between:

```text
Image layers
      ↓
Writable container layer
      ↓
Container filesystem
```

and why changes inside a container normally disappear when the container is removed.
