# Phase 1 — Part 7: Base Images

## Where we are

```text
Phase 1 — Docker Fundamentals

01. Container vs Image       ✅
02. Dockerfile               ✅
03. Build Context            ✅
04. Docker Image Layers      ✅
05. Docker Image Registry    ✅
06. Container Filesystem     ✅
07. Base Images              ← NOW
08. Image Tags
```

We have already used examples like:

```dockerfile
FROM alpine
```

and:

```dockerfile
FROM python:3.12
```

Now we'll understand exactly what `FROM` means and why choosing a base image matters.

---

# 1. What Is a Base Image?

A **base image is the starting point for building another Docker image.**

For example:

```dockerfile
FROM alpine
```

means:

> Start this image build from the `alpine` image.

Or:

```dockerfile
FROM python:3.12
```

means:

> Start from an image that already provides the Python 3.12 environment.

The basic relationship is:

```text
Base Image
    │
    │ FROM
    ▼
Your Dockerfile
    │
    │ additional instructions
    ▼
Your Application Image
```

---

# 2. Why Do We Need a Base Image?

An application normally needs more than just its own source code.

For example, a Python application needs:

```text
Python runtime
System libraries
Application dependencies
Application code
```

Instead of building all of that from nothing, we can start with:

```dockerfile
FROM python:3.12
```

Then add our application:

```dockerfile
COPY .
RUN ...
```

Conceptually:

```text
python:3.12
     │
     ├── Python runtime
     ├── OS/filesystem components
     └── supporting libraries
             │
             │ your Dockerfile
             ▼
       Your application
             │
             ▼
        Final image
```

This is one of the biggest advantages of base images: **we don't have to construct the entire runtime environment ourselves.**

---

# 3. `FROM` Is Usually the First Instruction

A typical Dockerfile starts with:

```dockerfile
FROM python:3.12
```

followed by instructions such as:

```dockerfile
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

The simplified flow is:

```text
FROM
 ↓
Start with base image

WORKDIR
 ↓
Set working directory

COPY
 ↓
Add application files

RUN
 ↓
Install/build things

CMD
 ↓
Define default startup command
```

So `FROM` establishes the **starting filesystem and environment** for the image build.

---

# 4. A Base Image Is Itself an Image

This is an important connection to Part 4.

When you write:

```dockerfile
FROM alpine
```

you're not getting some magical empty environment.

`alpine` is itself a Docker image.

Conceptually:

```text
alpine image
┌──────────────────────┐
│ Alpine layer(s)      │
└──────────────────────┘
          │
          │ FROM
          ▼
your image
┌──────────────────────┐
│ Your application     │
├──────────────────────┤
│ Alpine layers        │
└──────────────────────┘
```

Therefore:

> **A base image becomes part of the foundation of your final image.**

---

# 5. Base Image + Your Layers

Suppose:

```dockerfile
FROM alpine

RUN apk add --no-cache nginx

COPY index.html /usr/share/nginx/html/
```

Conceptually:

```text
Final Image
┌────────────────────────────────┐
│ COPY index.html                │
├────────────────────────────────┤
│ RUN apk add nginx              │
├────────────────────────────────┤
│ Alpine base image layers       │
└────────────────────────────────┘
```

This connects directly to what we learned in Part 4.

The base image provides existing layers, and your build adds additional content.

---

# 6. Common Types of Base Images

You will encounter several categories.

### Minimal Linux distribution

```dockerfile
FROM alpine
```

Small and commonly used for lightweight applications.

---

### General-purpose Linux distribution

```dockerfile
FROM ubuntu:24.04
```

Provides a more complete Ubuntu environment.

---

### Language/runtime image

```dockerfile
FROM python:3.12
```

or:

```dockerfile
FROM node:22
```

These are useful when your application requires a particular runtime.

---

### Web/server image

```dockerfile
FROM nginx
```

Provides an existing Nginx environment.

The correct choice depends on what your application actually needs.

---

# 7. "Minimal" Does Not Automatically Mean "Better"

A common beginner assumption is:

> Smaller image = always better.

That's not necessarily true.

For example:

```text
alpine
```

is smaller than many general-purpose distributions, but the smallest possible image isn't automatically the best choice for every application.

You should consider:

* required libraries
* compatibility
* security
* maintainability
* debugging requirements
* application runtime
* organizational standards

For example, an application may work perfectly with a Debian-based Python image but encounter compatibility issues with a more minimal base.

So the goal is not:

```text
smallest possible image
```

but:

```text
appropriate, secure, maintainable image
```

---

# 8. What Is a "Base" Image vs an "Empty" Image?

There is an important distinction.

Many applications can start with:

```dockerfile
FROM alpine
```

or:

```dockerfile
FROM ubuntu
```

But Docker also supports very minimal images such as:

```dockerfile
FROM scratch
```

`scratch` is special.

It represents an **empty starting point** rather than a normal Linux distribution.

Conceptually:

```text
FROM alpine

Your image
    ↓
Alpine filesystem
    +
Your application
```

while:

```text
FROM scratch

Your image
    ↓
Nothing initially
    +
Your application
```

`scratch` is useful for certain applications, especially statically compiled binaries, but it requires you to understand exactly what your application needs.

For our fundamentals, remember:

> **`scratch` is essentially an empty starting point, while images such as Alpine and Ubuntu provide an existing filesystem/environment.**

---

# 9. Base Image Selection and Security

Your base image becomes part of your application image.

Therefore, if the base image contains:

```text
OS packages
libraries
runtime components
```

those components become part of the resulting image environment.

That means base-image selection affects your security posture.

For example:

```text
Base Image
    │
    ├── OS packages
    ├── system libraries
    └── runtime
           │
           ▼
      Final Image
           │
           ▼
   Security scanning
```

This is why CI/CD pipelines often include image vulnerability scanning tools such as Trivy.

The scanner doesn't only care about your application code; vulnerabilities can exist in the underlying image components as well.

---

# 10. Base Image Updates

Suppose your Dockerfile says:

```dockerfile
FROM python:3.12
```

The base image may receive updates over time.

For example:

```text
Old base image
    ↓
Security updates
    ↓
New base image
```

If you rebuild your application using an updated base image, your resulting image may contain updated underlying packages.

This is another reason image builds need to be reproducible and managed carefully in CI/CD.

We'll later discuss the difference between:

```text
python:3.12
```

and more specific version references when we reach **Image Tags**.

---

# 11. Official Images and Other Sources

Base images can come from registries.

For example:

```dockerfile
FROM python:3.12
```

Docker needs to find the referenced image.

Conceptually:

```text
Dockerfile
   │
   │ FROM python:3.12
   ▼
Registry
   │
   ▼
python:3.12
   │
   ▼
Your image build
```

If no registry is explicitly specified, Docker commonly uses its configured/default registry behavior.

The important connection to Part 5 is:

```text
Registry
   ↓
Base Image
   ↓
Your Build
   ↓
Final Image
```

So registries don't only store your finished application images.

They can also provide the **base images used during builds**.

---

# 12. Base Images and CI/CD

Consider a Jenkins pipeline:

```text
Git Repository
      │
      ▼
Dockerfile
      │
      │ FROM python:3.12
      ▼
Base Image
      │
      ▼
Docker Build
      │
      ▼
Application Image
      │
      ▼
Registry
```

This means a CI pipeline may interact with registries in two directions:

```text
                 Registry
                ↗        ↘
        pull base       push application
             ↑              ↓
          CI Build ──────────
```

More explicitly:

```text
Registry
   │
   │ pull
   ▼
Base Image
   │
   ▼
Docker Build
   │
   ▼
Application Image
   │
   │ push
   ▼
Registry
```

This is an important CI/CD mental model.

---

# 13. Hands-on Exercise

Let's inspect a simple base-image relationship.

Create:

```text
base-demo/
└── Dockerfile
```

Use:

```dockerfile
FROM alpine

RUN echo "Hello from my image"
```

Build:

```bash
docker build -t base-demo:1.0 .
```

Now inspect:

```bash
docker history base-demo:1.0
```

You'll see your build history along with the underlying Alpine image history.

The conceptual structure is:

```text
base-demo:1.0
     │
     ├── RUN echo ...
     │
     └── Alpine base
```

Now inspect the base image itself:

```bash
docker history alpine
```

Compare the two outputs.

You should notice that your image is built **on top of** the Alpine image.

---

# 14. Hands-on Exercise — Compare Base Images

Create two Dockerfiles.

### Dockerfile.alpine

```dockerfile
FROM alpine

RUN echo "Hello"
```

### Dockerfile.ubuntu

```dockerfile
FROM ubuntu:24.04

RUN echo "Hello"
```

Build them:

```bash
docker build -f Dockerfile.alpine -t base-demo:alpine .
```

and:

```bash
docker build -f Dockerfile.ubuntu -t base-demo:ubuntu .
```

Check:

```bash
docker images
```

You'll likely see that the resulting images have different sizes.

You can also inspect them:

```bash
docker history base-demo:alpine
```

and:

```bash
docker history base-demo:ubuntu
```

The purpose of this exercise is not to memorize which one is "better."

It's to observe:

> **Changing the base image changes the foundation of your final image.**

---

# 15. Base Image vs Application Image

Keep these two concepts separate.

### Base image

Provides the starting environment:

```text
python:3.12
```

### Application image

Built by your Dockerfile:

```text
my-company/payment-service:1.0
```

Conceptually:

```text
python:3.12
     │
     │ FROM
     ▼
Dockerfile
     │
     │ application + dependencies
     ▼
payment-service:1.0
```

The application image is therefore **derived from** the base image.

---

# 16. Common Mistakes

### Mistake 1 — Thinking `FROM` downloads an operating system VM

It doesn't create a virtual machine.

A container image provides filesystem content and runtime components; containers normally share the host kernel.

This connects to the filesystem discussion from Part 6.

---

### Mistake 2 — Thinking Alpine is always the best base image

Smaller doesn't automatically mean better.

Choose based on application requirements, compatibility, security, and maintainability.

---

### Mistake 3 — Thinking the base image is separate from the final image

The base image's layers form part of the foundation of the resulting image.

```text
Base image
    +
Your layers
    =
Final image
```

---

### Mistake 4 — Ignoring the base image in vulnerability management

Your final image inherits components from its base image.

Therefore, the base image is part of your security and maintenance responsibility.

---

# 17. The Mental Model

At this point, combine Parts 4, 5, and 7:

```text
                 Registry
                    │
                    │ pull
                    ▼
               Base Image
                    │
                    │ FROM
                    ▼
                Dockerfile
                    │
                    │ build
                    ▼
             Application Image
                    │
                    │ push
                    ▼
                 Registry
                    │
                    │ pull
                    ▼
                Deployment
```

And at the image level:

```text
Application Image
┌──────────────────────────────┐
│ Your application layer       │
├──────────────────────────────┤
│ Your dependency layer        │
├──────────────────────────────┤
│ Base image layers            │
└──────────────────────────────┘
```

That's the core idea of a base image.

---

# 18. What You Should Know Now

You should understand:

* A **base image is the starting point for another image build**.
* `FROM` specifies the base image.
* A base image is itself a Docker image.
* The base image provides the foundation of the final image.
* Your Dockerfile adds application-specific content on top.
* Base images can be minimal, general-purpose, or runtime-specific.
* Smaller does not automatically mean better.
* `scratch` provides an essentially empty starting point.
* Base images affect image size, compatibility, maintenance, and security.
* CI pipelines typically **pull base images** and **push the resulting application image**.

The essential model is:

```text
Base Image
    │
    │ FROM
    ▼
Dockerfile
    │
    │ build
    ▼
Application Image
    │
    │ push
    ▼
Registry
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
07. Base Images              ✅
08. Image Tags               ← NEXT
```

We are now at the **final topic of Phase 1**.

Next we'll study **Image Tags**, including:

```text
nginx
nginx:latest
nginx:1.27
nginx:1.27.5
registry.example.com/team/app:1.0
```

and, importantly for CI/CD:

```text
Tag
  vs
Digest
  vs
Image ID
```

That will complete Phase 1 and give us the vocabulary needed for the actual image-building phase.
