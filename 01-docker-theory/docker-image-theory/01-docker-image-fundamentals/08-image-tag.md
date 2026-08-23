# Phase 1 — Part 8: Docker Image Tags

## Where we are

```text
Phase 1 — Docker Fundamentals

01. Container vs Image       ✅
02. Dockerfile               ✅
03. Build Context            ✅
04. Docker Image Layers      ✅
05. Docker Image Registry    ✅
06. Container Filesystem     ✅
07. Base Images              ✅
08. Image Tags               ← NOW
```

This is the **final topic of Phase 1**.

Image tags look simple:

```text
nginx:1.27
```

but they are extremely important in CI/CD because tags are how we commonly identify, publish, promote, and deploy image versions.

---

# 1. What Is an Image Tag?

A **tag is a human-readable name attached to an image reference**.

For example:

```text
nginx:1.27
```

can be understood as:

```text
nginx
  │
  └── repository

1.27
  │
  └── tag
```

Another example:

```text
mycompany/payment-service:2.4.1
```

means:

```text
mycompany/payment-service
        │
        └── repository

2.4.1
        │
        └── tag
```

A tag makes it easier for humans and tools to refer to an image.

---

# 2. Image Reference Structure

Let's build up a complete reference.

Consider:

```text
registry.example.com/myteam/payment-service:2.4.1
```

Conceptually:

```text
registry.example.com
        │
        └── Registry

myteam/payment-service
        │
        └── Repository

2.4.1
        │
        └── Tag
```

So a common image reference looks like:

```text
[registry/]repository[:tag]
```

For example:

```text
nginx:1.27
```

or:

```text
docker.io/myuser/myapp:1.0
```

or:

```text
harbor.example.com/devops/myapp:2026.08.24
```

---

# 3. What Happens If You Don't Specify a Tag?

Suppose you run:

```bash
docker pull nginx
```

You didn't explicitly provide a tag.

Docker uses:

```text
nginx:latest
```

as the default tag in the normal case.

So:

```bash
docker pull nginx
```

is effectively equivalent to:

```bash
docker pull nginx:latest
```

This is convenient for learning and interactive use.

But it introduces an important CI/CD concern.

---

# 4. Why `latest` Can Be Dangerous

Imagine your deployment uses:

```text
myapp:latest
```

Today:

```text
latest → Version A
```

Tomorrow:

```text
latest → Version B
```

The deployment reference didn't change:

```text
myapp:latest
```

but the actual image behind that tag did.

Conceptually:

```text
Day 1

myapp:latest
      ↓
   Version A


Day 2

myapp:latest
      ↓
   Version B
```

Therefore:

> **A tag is a mutable reference. It should not automatically be assumed to permanently identify one specific image.**

This is one of the most important lessons for CI/CD.

---

# 5. Tags Do Not Create New Images

Suppose:

```bash
docker build -t myapp:1.0 .
```

Then:

```bash
docker tag myapp:1.0 myapp:stable
```

You now have:

```text
myapp:1.0
myapp:stable
```

It may look like two images:

```text
myapp:1.0
myapp:stable
```

but both references can point to the **same underlying image**.

Conceptually:

```text
              ┌── myapp:1.0
              │
Docker Image ─┤
              │
              └── myapp:stable
```

`docker tag` doesn't rebuild the image.

It gives the image another reference.

---

# 6. Hands-on Exercise — Multiple Tags

Build:

```bash
docker build -t tag-demo:1.0 .
```

Check:

```bash
docker images
```

Then:

```bash
docker tag tag-demo:1.0 tag-demo:stable
```

Check again:

```bash
docker images
```

You may see:

```text
REPOSITORY   TAG
tag-demo     1.0
tag-demo     stable
```

Now inspect the image IDs:

```bash
docker images tag-demo
```

You'll see that the tags can reference the same image ID.

This demonstrates:

```text
Two tags
   ↓
Same image
```

---

# 7. Tags Are Often Used for Versioning

A common practice is to give images application versions.

For example:

```text
payment-service:1.0.0
payment-service:1.1.0
payment-service:2.0.0
```

This makes the image reference meaningful:

```text
payment-service:1.0.0
             ↓
       application version
```

A CI pipeline might produce:

```text
Build #101 → payment-service:1.0.0
Build #102 → payment-service:1.0.1
Build #103 → payment-service:1.0.2
```

This is much easier to reason about than repeatedly overwriting:

```text
payment-service:latest
```

---

# 8. Common CI/CD Tagging Strategies

There isn't one universal tagging strategy.

You'll encounter several.

### Version tag

```text
myapp:1.4.2
```

Good for release versions.

---

### Git commit/tag

```text
myapp:a83f91c
```

The tag can represent a Git commit.

This provides a direct relationship:

```text
Git commit
    ↓
Docker image
```

---

### CI build number

```text
myapp:build-105
```

Useful for identifying exactly which pipeline build produced the image.

---

### Environment tag

```text
myapp:dev
myapp:staging
myapp:production
```

These can be useful, but remember that these tags can move.

For example:

```text
production
    ↓
Version 10

later

production
    ↓
Version 11
```

So environment tags should not automatically be treated as immutable version identifiers.

---

# 9. One Image Can Have Many Tags

Suppose we have one image:

```text
Image X
```

It could have:

```text
myapp:1.5.0
myapp:stable
myapp:production
```

all pointing to the same image.

Conceptually:

```text
                    ┌── myapp:1.5.0
                    │
Image X ────────────┼── myapp:stable
                    │
                    └── myapp:production
```

This is useful when promoting an image through environments.

For example:

```text
Build
  ↓
myapp:1.5.0
  ↓
Testing
  ↓
Stable
  ↓
Production
```

But again, the tags such as `stable` and `production` can be moved to another image later.

---

# 10. Tag vs Image ID

Now we need to distinguish another term.

Run:

```bash
docker images
```

You might see:

```text
REPOSITORY   TAG    IMAGE ID
myapp        1.0    abc123...
```

Here:

```text
myapp:1.0
```

is a human-friendly image reference.

```text
abc123...
```

is the local Docker **image ID**.

These are not the same thing.

Conceptually:

```text
myapp:1.0
    │
    └── tag/reference
          │
          ▼
      Image object
          │
          └── Image ID
```

The image ID identifies the local image object in Docker's image store.

---

# 11. Tag vs Digest

This distinction is even more important.

An image can be referenced using a tag:

```text
myapp:1.0
```

or by digest:

```text
myapp@sha256:abcdef123...
```

Think of them as:

```text
Tag
 │
 └── human-friendly, mutable reference


Digest
 │
 └── content-addressed identity
```

A digest looks something like:

```text
sha256:9f2c...
```

The full digest is much longer.

---

# 12. Why Digests Matter

Suppose:

```text
myapp:production
```

currently points to:

```text
Image A
```

Later someone updates the tag:

```text
myapp:production
        ↓
     Image B
```

If you deploy using:

```text
myapp:production
```

you may get a different image at a later time.

But if you deploy using:

```text
myapp@sha256:...
```

you're identifying a specific image manifest by its digest.

This is much better for reproducibility.

Conceptually:

```text
Tag:

myapp:production
       ↓
     Image ?
       ↓
can change


Digest:

myapp@sha256:ABC...
       ↓
specific content
```

---

# 13. Tag vs Digest vs Image ID

This is worth memorizing.

| Concept  | Example               | Purpose                       |
| -------- | --------------------- | ----------------------------- |
| Tag      | `myapp:1.0`           | Human-friendly reference      |
| Digest   | `myapp@sha256:abc...` | Content-based image identity  |
| Image ID | `abc123...`           | Local Docker image identifier |

The important distinction is:

```text
Tag
  ↓
Can move


Digest
  ↓
Identifies specific content


Image ID
  ↓
Local Docker image-store identifier
```

---

# 14. How Tags Work With a Registry

Suppose you build:

```bash
docker build -t myapp:1.0 .
```

Then you tag it for your registry:

```bash
docker tag myapp:1.0 \
  registry.example.com/team/myapp:1.0
```

Then:

```bash
docker push registry.example.com/team/myapp:1.0
```

The flow becomes:

```text
Local Image
     │
     ├── myapp:1.0
     │
     └── registry.example.com/team/myapp:1.0
                    │
                    │ push
                    ▼
                 Registry
```

Again, tagging does not mean rebuilding.

---

# 15. A CI/CD Example

Suppose Jenkins builds commit:

```text
a83f91c
```

The pipeline could create:

```text
myapp:a83f91c
```

and perhaps:

```text
myapp:build-105
```

After testing, the same image could receive:

```text
myapp:production
```

Conceptually:

```text
                    ┌── myapp:a83f91c
                    │
Built Image ────────┼── myapp:build-105
                    │
                    └── myapp:production
```

All three could point to the same image.

This creates a useful separation:

```text
Version/identity tag
        +
Environment tag
```

For example:

```text
myapp:a83f91c
myapp:production
```

The first tells you **which build**.

The second tells you **where that build is currently promoted**.

---

# 16. Why `latest` Is Often Misunderstood

People sometimes think:

```text
latest = newest image
```

That's not quite the right mental model.

`latest` is simply a **tag name**.

It doesn't inherently mean:

```text
highest version
```

or:

```text
most recently created
```

It is a conventional tag that can be assigned to whichever image a publisher chooses.

So:

```text
myapp:latest
```

doesn't guarantee:

```text
highest version
```

Think:

```text
latest
  ↓
just another tag
```

---

# 17. Hands-on Exercise — Inspect Tags

Run:

```bash
docker images
```

Find an image you have built.

For example:

```text
REPOSITORY   TAG
tag-demo     1.0
```

Create another tag:

```bash
docker tag tag-demo:1.0 tag-demo:test
```

Then:

```bash
docker images tag-demo
```

You should see:

```text
REPOSITORY   TAG
tag-demo     1.0
tag-demo     test
```

Now remove one tag:

```bash
docker rmi tag-demo:test
```

The underlying image doesn't necessarily disappear because another tag still references it.

Then:

```bash
docker images tag-demo
```

You should still see:

```text
tag-demo:1.0
```

This demonstrates that tags are references to images rather than separate image contents.

---

# 18. Tagging and Reproducibility

This is particularly important for CI/CD.

Consider:

```text
Dockerfile
    ↓
docker build
    ↓
myapp:latest
```

Six months later, you might not know exactly which image `latest` represented at the time of deployment.

Compare that with:

```text
Dockerfile
    ↓
docker build
    ↓
myapp:1.4.2
    ↓
digest: sha256:...
```

Now you have much stronger traceability.

A mature CI/CD pipeline often records:

```text
Source commit
     ↓
CI build
     ↓
Image tag
     ↓
Image digest
     ↓
Deployment
```

That relationship becomes extremely valuable when troubleshooting production deployments.

---

# 19. What Happens During `docker pull`?

Suppose:

```bash
docker pull nginx:1.27
```

Docker asks the registry for the image associated with that reference.

Conceptually:

```text
nginx:1.27
    │
    ▼
Registry
    │
    ▼
Image manifest
    │
    ├── Layer 1
    ├── Layer 2
    ├── Layer 3
    └── ...
```

Docker then downloads whatever content it doesn't already have locally.

This connects directly back to Part 4:

```text
Tag
 ↓
Image manifest
 ↓
Image layers
 ↓
Local image
```

---

# 20. A More Complete Image Mental Model

We can now connect almost the entire Phase 1.

Consider:

```text
registry.example.com/team/payment-service:1.2.0
```

The reference identifies an image in a registry.

That image consists conceptually of:

```text
Image
┌──────────────────────────────┐
│ Application layer            │
├──────────────────────────────┤
│ Dependency layer             │
├──────────────────────────────┤
│ Base image layers            │
└──────────────────────────────┘
```

The image has:

```text
Tag:
1.2.0

Digest:
sha256:....

Image ID:
local Docker identifier
```

And when pulled and run:

```text
Image
  │
  │ docker run
  ▼
Container
  │
  └── Writable container layer
```

This is the complete mental model we've been building throughout Phase 1.

---

# 21. Common Mistakes

### Mistake 1 — Thinking a tag is a version

A tag **can represent a version**, but a tag itself is simply a reference.

```text
1.0.0
```

is a tag name.

It doesn't automatically guarantee immutability.

---

### Mistake 2 — Thinking `latest` always means newest

It doesn't.

```text
latest
```

is just a tag.

---

### Mistake 3 — Thinking `docker tag` creates a new image

It doesn't normally create a separate copy of the image contents.

It creates another reference to the image.

---

### Mistake 4 — Thinking tags are immutable

Tags can be moved.

For example:

```text
myapp:production
       ↓
Image A
```

can later become:

```text
myapp:production
       ↓
Image B
```

---

### Mistake 5 — Confusing Image ID and Digest

They serve different purposes.

```text
Image ID
    ↓
Local Docker image store


Digest
    ↓
Content-addressed image identity
```

---

# 22. What You Should Know Now

You should now understand:

* A tag is a human-readable reference to an image.
* An image reference can include a registry, repository, and tag.
* `latest` is simply a conventional tag.
* Tags can be changed or moved.
* `docker tag` creates another reference to an image.
* One image can have multiple tags.
* Tags are commonly used for application versions and CI build identifiers.
* Image IDs identify local Docker images.
* Digests provide content-based image identification.
* Digests are useful for reproducible deployments.
* Tags and digests serve different purposes.

The essential model is:

```text
Registry
   │
   ▼
Repository
   │
   ├── :1.0
   ├── :1.1
   ├── :stable
   └── :latest
          │
          ▼
       Image
          │
          ├── Image layers
          │
          └── Digest
```

---

# Phase 1 — COMPLETE 🎯

We've now finished the entire planned Phase 1:

```text
Phase 1 — Docker Fundamentals

01. Container vs Image       ✅
02. Dockerfile               ✅
03. Build Context            ✅
04. Docker Image Layers      ✅
05. Docker Image Registry    ✅
06. Container Filesystem     ✅
07. Base Images              ✅
08. Image Tags               ✅
```

More importantly, the concepts now connect:

```text
                    Dockerfile
                        │
                        │ uses
                        ▼
                  Build Context
                        │
                        ▼
                    Base Image
                        │
                        ▼
                  Build Process
                        │
                        ▼
                   Image Layers
                        │
                        ▼
                   Docker Image
                   /         \
                  /           \
             Tag             Digest
                \             /
                 \           /
                  ▼         ▼
                    Registry
                        │
                        │ pull
                        ▼
                     Container
                        │
                        ▼
              Writable Filesystem
```

### The CI/CD mental model

And this is the bigger picture you should carry forward:

```text
Developer
    │
    │ source code
    ▼
Git Repository
    │
    ▼
CI Pipeline
    │
    │ docker build
    ▼
Docker Image
    │
    ├── Layers
    ├── Tag
    └── Digest
    │
    │ docker push
    ▼
Container Registry
    │
    │ docker pull
    ▼
Deployment Environment
    │
    ▼
Container
```

That is the foundation we needed before moving into **actual standard Docker image building**.

## Next Phase

```text
Phase 1 — Docker Fundamentals       ✅ COMPLETE

                    ↓

Phase 2 — Build Your First Docker Image
```

In Phase 2, we stop primarily discussing individual concepts and start putting them together into **real image-building workflows**: project structure → Dockerfile → build context → `docker build` → inspecting the result → running it → iterating on the image.
