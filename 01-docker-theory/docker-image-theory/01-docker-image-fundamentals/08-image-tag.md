# Phase 1 — Part 7: Docker Base Images

We are now moving to:

```text
01. Container vs Image       ✅
02. Dockerfile               ✅
03. Build context            ✅
04. Docker image layers      ✅
05. Docker image registry    ✅
06. Container filesystem     ✅
07. Base images              ← NOW
08. Image tags
```

This is a very important step because it answers the question we ended Step 6 with:

> **Where does the initial filesystem inside a Docker image come from?**

The answer starts with one Dockerfile instruction:

```dockerfile
FROM
```

---

# 1. Start with the basic concept

Look at this Dockerfile:

```dockerfile
FROM ubuntu:24.04

COPY app.py /app/app.py
```

The first line says:

```dockerfile
FROM ubuntu:24.04
```

This means:

> **Start building my new image from the existing `ubuntu:24.04` image.**

So instead of creating the entire filesystem from nothing, Docker starts with an existing image.

Conceptually:

```text
ubuntu:24.04
      |
      | FROM
      v
Your image
```

That existing image is the **base image** for your build.

---

# 2. Why do we need a base image?

Remember the container filesystem from Step 6.

A typical Linux container filesystem looks something like:

```text
/
├── bin/
├── dev/
├── etc/
├── home/
├── lib/
├── usr/
├── var/
└── ...
```

Creating all of that manually would be ridiculous.

Instead, Docker can start from an existing filesystem:

```text
Ubuntu image
      |
      v
Base filesystem
      |
      +-- your application
      +-- your dependencies
      +-- your configuration
```

So a base image gives your Docker build a starting point.

---

# 3. The mental model

Think of a base image as the **foundation of your image**.

For example:

```text
             Your Application Image
                     │
          ┌──────────┴──────────┐
          │ Your application    │
          ├─────────────────────┤
          │ Your dependencies   │
          ├─────────────────────┤
          │ Ubuntu base image   │
          └─────────────────────┘
```

Or:

```text
Base Image
    ↓
Add dependencies
    ↓
Add application
    ↓
Add configuration
    ↓
Your final image
```

This is very similar to building a house.

```text
Foundation
    ↓
Structure
    ↓
Rooms
    ↓
Furniture
```

The base image is the foundation.

---

# 4. `FROM` is the key instruction

Consider:

```dockerfile
FROM ubuntu:24.04
```

The syntax is:

```text
FROM <image>
```

For example:

```dockerfile
FROM ubuntu
```

or:

```dockerfile
FROM ubuntu:24.04
```

or:

```dockerfile
FROM python:3.12
```

or:

```dockerfile
FROM node:22
```

or:

```dockerfile
FROM alpine:3.22
```

Each one says:

> Start this image build from the specified existing image.

---

# 5. Example: Ubuntu base image

Suppose you write:

```dockerfile
FROM ubuntu:24.04

RUN apt-get update
RUN apt-get install -y nginx
```

Conceptually:

```text
ubuntu:24.04
      │
      │ base
      ▼
+----------------+
| Ubuntu         |
+----------------+
      │
      │ RUN apt
      ▼
+----------------+
| Ubuntu + nginx |
+----------------+
```

Your final image is no longer simply Ubuntu.

It is:

```text
Ubuntu base
+
nginx
```

---

# 6. Example: Python base image

Now consider:

```dockerfile
FROM python:3.12

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY app.py /app/app.py
```

The base image already provides a Python environment.

Conceptually:

```text
python:3.12
     |
     +-- Linux filesystem
     +-- Python
     +-- Python runtime environment
     |
     v
Install dependencies
     |
     v
Copy application
     |
     v
Final application image
```

This is extremely common.

Instead of manually doing:

```text
Install Linux
Install Python
Configure Python
Install pip
...
```

you start with:

```dockerfile
FROM python:3.12
```

and build from there.

---

# 7. Example: Node.js

For a Node.js application:

```dockerfile
FROM node:22

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

CMD ["npm", "start"]
```

The base image provides the environment needed to run Node.js.

Conceptually:

```text
node:22
   ↓
Node.js environment
   ↓
npm dependencies
   ↓
application code
   ↓
your final image
```

---

# 8. So what exactly is a base image?

A base image is simply:

> **The image specified as the starting point of another image build.**

For example:

```dockerfile
FROM python:3.12
```

Here:

```text
python:3.12
```

is the base image.

If you write:

```dockerfile
FROM ubuntu:24.04
```

then:

```text
ubuntu:24.04
```

is the base image.

If you write:

```dockerfile
FROM node:22
```

then:

```text
node:22
```

is the base image.

The term is **relative to your build**.

---

# 9. A very important distinction

Don't think:

> "A base image is always Ubuntu."

No.

Ubuntu is just one possible base.

You can have:

```text
Ubuntu
Debian
Alpine
Rocky Linux
Amazon Linux
Python
Node
Java
Nginx
...
```

as starting points.

And there's an even deeper concept:

> **A base image doesn't necessarily have to be a traditional Linux distribution image.**

We'll get there shortly.

---

# 10. Base image vs final image

Suppose:

```dockerfile
FROM ubuntu:24.04

RUN apt-get update && apt-get install -y nginx

COPY index.html /var/www/html/
```

Here:

```text
ubuntu:24.04
```

is the **base image**.

The resulting image:

```text
my-web-server:1.0
```

is your **final image**.

So:

```text
Base Image
    +
Dockerfile instructions
    =
Final Image
```

More specifically:

```text
ubuntu:24.04
      |
      +-- RUN apt install nginx
      |
      +-- COPY index.html
      |
      v
my-web-server:1.0
```

---

# 11. Base images and image layers

Now we can connect this directly to Step 4.

Suppose the base image contains:

```text
ubuntu:24.04

Layer 3
Layer 2
Layer 1
```

Your Dockerfile says:

```dockerfile
FROM ubuntu:24.04

RUN apt-get update
RUN apt-get install -y nginx

COPY index.html /var/www/html/
```

Docker builds additional layers on top:

```text
Your final image

Layer 6 → COPY index.html
Layer 5 → install nginx
Layer 4 → apt update
Layer 3 → Ubuntu base
Layer 2 → Ubuntu base
Layer 1 → Ubuntu base
```

So:

```text
Base image layers
        +
Your new layers
        =
Final image
```

This is why understanding image layers **before** base images was important.

---

# 12. The `FROM` instruction is therefore more important than it looks

When you see:

```dockerfile
FROM ubuntu:24.04
```

don't think only:

> "Use Ubuntu."

Think:

```text
Find the referenced image
        ↓
Use its filesystem/layers as the starting point
        ↓
Apply my Dockerfile instructions
        ↓
Create new layers
        ↓
Produce my final image
```

That's the deeper meaning.

---

# 13. Where does Docker get the base image?

Suppose you write:

```dockerfile
FROM ubuntu:24.04
```

Where does Docker find it?

Docker first needs the image locally or needs to obtain it from a registry.

Conceptually:

```text
Dockerfile
    |
    v
FROM ubuntu:24.04
    |
    v
Do I have this image locally?
    |
   no
    |
    v
Pull from registry
    |
    v
Ubuntu image available locally
    |
    v
Continue build
```

This connects directly to Step 5.

Remember:

```text
Registry
    |
    | pull
    v
Base image
    |
    v
Docker build
```

So registries are also extremely important for base images.

---

# 14. Example of a first build

Suppose you have:

```dockerfile
FROM ubuntu:24.04

RUN apt-get update
RUN apt-get install -y curl
```

You run:

```bash
docker build -t my-ubuntu:1.0 .
```

If `ubuntu:24.04` isn't available locally, Docker needs to retrieve it.

Conceptually:

```text
                  Registry
                     |
                     | pull
                     v
              ubuntu:24.04
                     |
                     v
                Docker Build
                     |
                     v
              my-ubuntu:1.0
```

This is something you'll see in real life when running `docker build`.

---

# 15. Why not build everything from scratch?

You could theoretically construct an image filesystem yourself.

But normally you don't want to.

Imagine creating:

```text
/etc
/usr
/bin
/lib
...
```

and manually installing everything needed.

Instead:

```dockerfile
FROM ubuntu:24.04
```

gives you a known starting point.

Then:

```dockerfile
RUN ...
COPY ...
ENV ...
WORKDIR ...
CMD ...
```

customize it.

This provides:

* faster development
* standardized environments
* reusable foundations
* easier maintenance
* predictable builds

---

# 16. A very important concept: base image inheritance

Images can effectively build on other images.

For example:

```text
ubuntu
   ↓
python image
   ↓
your application image
```

Conceptually:

```text
Ubuntu
  |
  v
Python environment
  |
  v
Your application
```

Or:

```text
Debian
  |
  v
Python
  |
  v
Django application
```

Your application image doesn't need to independently recreate everything underneath.

It builds on the layers already provided by its base.

---

# 17. `python` is not necessarily a Linux distribution

This is an important beginner point.

When you see:

```dockerfile
FROM python:3.12
```

you might think:

> "Python is the base operating system."

No.

The Python image itself is built on top of another underlying image.

Conceptually:

```text
Your Django image
        |
        v
python:3.12
        |
        v
Debian / another base
        |
        v
Underlying filesystem
```

The exact underlying base depends on the specific Python image variant.

So image inheritance can form a chain.

---

# 18. Base image chains

Imagine:

```text
Your application
       |
       v
python:3.12
       |
       v
debian
       |
       v
base filesystem
```

Your final image therefore indirectly inherits the lower layers.

You can think of it as:

```text
Application layers
       ↓
Python layers
       ↓
Debian layers
       ↓
Base filesystem
```

This is why a seemingly simple:

```dockerfile
FROM python:3.12
```

can bring in quite a lot of existing image content.

---

# 19. What is a "minimal" base image?

You will often hear:

> "Use a minimal base image."

This means the image contains only what is reasonably necessary for its purpose.

For example, you may encounter:

```text
Ubuntu
Debian slim
Alpine
Distroless
Scratch
```

They represent different approaches to how much environment you include.

Conceptually:

```text
More general
      │
      v
Ubuntu
      ↓
Debian slim
      ↓
Alpine
      ↓
Distroless
      ↓
Scratch
      │
      v
More minimal
```

This is simplified because these are not simply points on one universal scale, but it's useful as an initial mental model.

---

# 20. Why would you want a smaller base?

Suppose:

```text
Image A = 1.2 GB
Image B = 180 MB
```

If both provide everything your application actually needs, the smaller image can have advantages:

* faster image pulls
* faster deployments
* less storage
* smaller attack surface
* potentially faster CI/CD pipelines

For example:

```text
CI
 ↓
Build image
 ↓
Push image
 ↓
Registry
 ↓
Kubernetes pulls image
```

A smaller image can reduce the amount of data moved through that pipeline.

But:

> **Smaller is not automatically better.**

This is an important point.

---

# 21. Beginner misconception: "Smallest image is always best"

Not necessarily.

Suppose you choose an extremely minimal image but your application needs:

```text
shell
certificates
debugging tools
runtime libraries
system utilities
```

You may make development and troubleshooting much harder.

So the real goal is:

> **Use an appropriately minimal image that contains everything the application actually needs.**

Not:

> "Make the image as tiny as possible at any cost."

---

# 22. Ubuntu vs Alpine

You'll often see comparisons like:

```dockerfile
FROM ubuntu:24.04
```

versus:

```dockerfile
FROM alpine:3.22
```

The important point isn't simply:

```text
Ubuntu = big
Alpine = small
```

There are deeper differences involving:

* package managers
* libc implementation
* available system utilities
* compatibility
* debugging experience
* application dependencies

For example, Alpine commonly uses:

```text
musl libc
```

where many traditional Linux environments use:

```text
glibc
```

This can matter for some applications and binaries.

We don't need to go deep into that yet.

---

# 23. `scratch` — the special case

You may eventually see:

```dockerfile
FROM scratch
```

This is very different.

`scratch` is an empty starting point.

Conceptually:

```text
FROM scratch
      |
      v
Nothing
      |
      +-- your application files
      |
      v
Final image
```

There isn't a normal Linux distribution environment sitting underneath it.

This is useful for certain applications, particularly statically compiled applications such as some Go programs.

For example, conceptually:

```dockerfile
FROM scratch

COPY my-app /my-app

ENTRYPOINT ["/my-app"]
```

The resulting image can be extremely minimal.

But don't jump to `scratch` just because it's small.

It can make debugging and runtime functionality more difficult.

---

# 24. What is a "base image" really?

Here's a subtle but important distinction.

People often casually say:

> "Ubuntu is a base image."

That's fine.

But technically, **any image can serve as the base for another image**.

For example:

```dockerfile
FROM my-company-base:1.0
```

Now:

```text
my-company-base:1.0
```

is the base image for your new image.

So a base image isn't necessarily something special created by Docker.

It's simply:

> **The starting image referenced by `FROM`.**

---

# 25. Company base images

This becomes very important in production environments.

A company might create:

```text
company/java-base:21
```

which contains:

```text
Linux base
+
Java 21
+
CA certificates
+
timezone configuration
+
security configuration
+
company standards
```

Application teams then use:

```dockerfile
FROM company/java-base:21

COPY application.jar /app/application.jar

CMD ["java", "-jar", "/app/application.jar"]
```

Now many applications share the same organizational foundation.

Conceptually:

```text
                company/java-base:21
                    /     |     \
                   /      |      \
                  v       v       v
              App A    App B    App C
```

This is a very common enterprise pattern.

---

# 26. Base image and security

Base images are also a security concern.

Suppose your application uses:

```dockerfile
FROM some-old-image
```

and that base image contains vulnerable packages.

Your application image inherits those vulnerabilities.

Conceptually:

```text
Vulnerable base image
        |
        v
Your application image
        |
        v
Potential vulnerabilities
```

Therefore production teams need to:

* choose trusted base images
* keep them updated
* scan images
* track vulnerabilities
* rebuild applications when base images receive security updates

This connects directly to tools such as **Trivy**, which you'll eventually use in your CI/CD pipeline.

---

# 27. Base image updates don't automatically update your image

This is an important CI/CD concept.

Suppose today you build:

```dockerfile
FROM python:3.12
```

Your resulting image contains the base image content that existed when you built it.

If the `python:3.12` image later gets updated, your already-built image does **not magically change**.

You need to rebuild.

Conceptually:

```text
Old build
    |
    v
my-app:1.0
    |
    X
doesn't automatically change
```

Later:

```text
Updated base image
        |
        v
docker build
        |
        v
new application image
```

This is one reason CI/CD pipelines frequently rebuild images.

---

# 28. The base image is part of your supply chain

Think about your Dockerfile:

```dockerfile
FROM python:3.12
```

You may have written only one line, but you're depending on:

```text
python:3.12
    ↓
underlying base
    ↓
OS packages
    ↓
libraries
    ↓
your application
```

So your application image isn't only your own code.

It contains a chain of dependencies.

This is called part of the **software supply chain**.

Later, when we study:

```text
Trivy
SBOM
image signing
image provenance
```

this will become much more important.

---

# 29. What Docker is actually doing during `FROM`

Let's make the entire process concrete.

Suppose:

```dockerfile
FROM python:3.12

COPY app.py /app/app.py
```

You execute:

```bash
docker build -t my-app:1.0 .
```

Conceptually:

```text
Step 1
Docker reads:
FROM python:3.12

        ↓

Step 2
Docker finds python:3.12 locally
or pulls it from a registry

        ↓

Step 3
Docker uses the base image layers

        ↓

Step 4
Docker processes:
COPY app.py /app/app.py

        ↓

Step 5
Docker creates another image layer

        ↓

Step 6
Final image:
my-app:1.0
```

So:

```text
python:3.12
       +
COPY app.py
       =
my-app:1.0
```

---

# 30. The complete mental model of base images

At this point, you should be able to visualize:

```text
                 REGISTRY
                    |
                    | pull
                    v
             Base Image
            python:3.12
                    |
                    v
          ┌─────────────────┐
          │ Base layers     │
          │                 │
          │ Linux filesystem│
          │ Python runtime  │
          │ etc.            │
          └─────────────────┘
                    |
             Dockerfile
                    |
             COPY / RUN ...
                    |
                    v
          ┌─────────────────┐
          │ Your new layers │
          └─────────────────┘
                    |
                    v
             Final Image
                    |
                    v
                Container
```

This connects Steps 4, 5, 6, and 7 together.

---

# 31. Hands-on exercise

Let's make this concrete.

Create a simple Dockerfile:

```dockerfile
FROM ubuntu:24.04

RUN apt-get update && \
    apt-get install -y curl

CMD ["bash"]
```

Build it:

```bash
docker build -t base-demo:1.0 .
```

Then inspect:

```bash
docker images
```

Run it:

```bash
docker run -it base-demo:1.0
```

Inside:

```bash
curl --version
```

You should see that `curl` is available.

Why?

Because:

```text
ubuntu:24.04
       ↓
RUN apt install curl
       ↓
base-demo:1.0
```

The base image supplied the initial filesystem, and your Dockerfile added another layer containing the changes.

---

# 32. One more useful experiment

Run:

```bash
docker history base-demo:1.0
```

This is a useful command for connecting today's lesson to our previous **image layers** lesson.

You'll see the history of the image layers/instructions.

Conceptually:

```text
base-demo:1.0
       |
       +-- CMD
       +-- RUN apt install curl
       +-- FROM ubuntu
```

This helps you see that your final image is built from an existing image plus your additional build steps.

---

# 33. Beginner concepts vs advanced concepts

### You should understand now

You should be comfortable with:

* `FROM` specifies the starting image.
* That starting image is the base image.
* The base image provides the initial filesystem/layers.
* Your Dockerfile adds new layers on top.
* Base images can come from registries.
* A base image can itself be based on another image.
* `python`, `node`, etc. images aren't necessarily operating systems themselves.
* Smaller images can be useful, but smallest isn't automatically best.
* `scratch` is an empty starting point.
* Base images affect your application's security and dependency chain.
* Updating a base image requires rebuilding your application image to incorporate the update.

### Advanced — leave for later

Don't worry about these yet:

* OCI image specification
* manifest lists
* multi-architecture images
* digest pinning
* reproducible base-image builds
* distroless internals
* glibc vs musl details
* package-layer optimization
* image provenance
* SBOM generation
* cosign/signatures
* base-image lifecycle management

We'll encounter these naturally later.

---

# 34. The one sentence to remember

If you remember only one thing from Step 7:

> **A base image is the starting image specified by `FROM`; Docker uses its existing filesystem/layers as the foundation and adds your Dockerfile's changes on top to create the final image.**

The complete progression is now:

```text
Dockerfile
    ↓
FROM base image
    ↓
Base image layers
    ↓
Your Dockerfile instructions
    ↓
Additional layers
    ↓
Final Docker image
    ↓
Registry
    ↓
Container
    ↓
Container filesystem
```

And that naturally brings us to the final topic of Phase 1:

```text
08. Image tags
```

because we've repeatedly written things like:

```text
ubuntu:24.04
python:3.12
my-app:1.0
```

The `:24.04`, `:3.12`, and `:1.0` portions are **image tags**.
