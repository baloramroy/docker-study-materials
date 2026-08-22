# Phase 1 — Part 5: Docker Image Registry

We are now moving to:

```text
01. Container vs Image       ✅
02. Dockerfile               ✅
03. Build context             ✅
04. Docker image layers       ✅
05. Docker image registry     ← NOW
06. Container filesystem
07. Base images
08. Image tags
```

The goal of this lesson is **not** to learn Docker Hub commands first. The goal is to understand **what an image registry actually is, why it exists, and how Docker uses it**.

---

# 1. Start with the basic concept

So far, we've established:

* A **Dockerfile** describes how to build an image.
* The **build context** provides files to the build.
* Docker builds the image from **layers**.
* The resulting image exists on the machine where you built it.

Now imagine this:

```text
Your laptop
    |
    | docker build
    v
Docker Image
```

You successfully built:

```text
my-app:1.0
```

That image is currently available **on your laptop**.

But your production server is somewhere else.

```text
Developer Laptop                 Production Server

my-app:1.0                       ????
    |
    |
    X  ← image is not here
```

How does the production server get the image?

This is where an **image registry** comes in.

---

# 2. What is a Docker image registry?

A **Docker image registry is a server/service that stores and distributes container images.**

Think of it as a central storage system for Docker images.

Conceptually:

```text
                 Docker Image Registry
                         |
              +----------+----------+
              |          |          |
           image A    image B    image C
              |
          layers...
```

Examples include:

* Docker Hub
* Harbor
* Amazon Elastic Container Registry (ECR)
* GitHub Container Registry (GHCR)
* Google Artifact Registry
* Azure Container Registry (ACR)

The important concept is:

> **A registry is where container images can be stored so that other machines can pull them.**

---

# 3. The mental model

Think about Git.

You have a local Git repository:

```text
Developer Machine
      |
      v
   Git Repo
```

You can push it to GitHub:

```text
Developer Machine
      |
      | git push
      v
    GitHub
      |
      | git clone / pull
      v
Other Machine
```

Docker has a very similar concept.

```text
Developer Machine
      |
      | docker build
      v
Docker Image
      |
      | docker push
      v
Image Registry
      |
      | docker pull
      v
Server
```

This is one of the most important mental models in Docker.

---

# 4. Build vs Store vs Run

Don't mix these three activities.

### Build

Docker creates an image:

```bash
docker build -t my-app:1.0 .
```

Conceptually:

```text
Dockerfile
    +
Build Context
    |
    v
Docker Build
    |
    v
Image
```

---

### Store

The image can be uploaded to a registry:

```bash
docker push my-app:1.0
```

Conceptually:

```text
Local Image
    |
    | push
    v
Registry
```

---

### Run

Another machine can obtain the image:

```bash
docker pull my-app:1.0
```

and then run it:

```bash
docker run my-app:1.0
```

Conceptually:

```text
Registry
    |
    | pull
    v
Local Image
    |
    | run
    v
Container
```

So:

```text
BUILD
  ↓
IMAGE
  ↓
PUSH
  ↓
REGISTRY
  ↓
PULL
  ↓
IMAGE
  ↓
RUN
  ↓
CONTAINER
```

Keep this sequence in your head.

---

# 5. Why can't we just copy the image?

A beginner might think:

> "If I build the image on my laptop, why don't I just copy it to the server?"

You actually **can**.

For example:

```bash
docker save my-app:1.0 -o my-app.tar
```

Then transfer the file:

```text
Laptop
   |
   | my-app.tar
   v
Server
```

Then load it:

```bash
docker load -i my-app.tar
```

This works.

But it becomes inconvenient when you have:

```text
10 servers
50 servers
100 servers
Kubernetes cluster
multiple environments
multiple applications
```

A registry solves this distribution problem.

Instead of manually transferring image files:

```text
Laptop
  |
  +----> Server 1
  +----> Server 2
  +----> Server 3
  +----> Server 4
  +----> Server 5
```

you have:

```text
              Registry
             /    |    \
            /     |     \
           v      v      v
       Server1 Server2 Server3
```

---

# 6. What actually gets stored?

This is where our previous lesson about **image layers** becomes important.

Suppose we have:

```text
my-app:1.0
```

The image consists of layers:

```text
Layer 4: Application files
Layer 3: Dependencies
Layer 2: Runtime
Layer 1: Base filesystem
```

When you push the image:

```bash
docker push my-app:1.0
```

Docker does **not simply upload one giant mysterious image file**.

The registry stores the image's content, including its layers and metadata.

Conceptually:

```text
                 Registry
                    |
          +---------+---------+
          |         |         |
       Layer 1   Layer 2   Layer 3
          |         |         |
          +---------+---------+
                    |
                Layer 4
```

This connects directly to our previous lesson.

### Important connection

**Docker image layers are one reason registries can distribute images efficiently.**

If a layer already exists in the registry, Docker doesn't need to upload another identical copy.

---

# 7. A concrete example

Suppose you build:

```dockerfile
FROM python:3.12

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY app.py .
```

You build:

```bash
docker build -t my-app:1.0 .
```

Now your local Docker engine has:

```text
my-app:1.0
```

You want to store it in Docker Hub.

You would normally give it a registry-qualified name such as:

```text
username/my-app:1.0
```

Then:

```bash
docker tag my-app:1.0 username/my-app:1.0
```

and:

```bash
docker push username/my-app:1.0
```

Now the registry contains the image.

Another machine can do:

```bash
docker pull username/my-app:1.0
```

Then:

```bash
docker run username/my-app:1.0
```

The flow is:

```text
Developer Machine

Dockerfile
    |
    v
docker build
    |
    v
my-app:1.0
    |
    | docker push
    v
+----------------------+
|    Image Registry    |
|                      |
| username/my-app:1.0 |
+----------------------+
          |
          | docker pull
          v
Production Server
          |
          v
       Container
```

---

# 8. What is Docker Hub?

Docker provides **Docker Hub**, a public registry service.

![Image](https://images.openai.com/static-rsc-4/Lmk9577TeXT9dkZdGmOxxEVj6sHsc9EJo5P8Thj4pp2dbvPlfPNUImXkZ213mx6_R_NNL2wn8C5Id-n7xLupfVhRpzaMLoNfyza3o_tYiS1klOjMIBxPeHSl9mS2EShva69H5l6LIGsQdPhhLP6l2GX6GTsMFn-h-bde9zZTnv4rarYxHBFtO-ddVeCn6RUi?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/fBe66pqEJ2_dkPZRFyS_kXpwf6CeeUhqphdG4XpDrRYPJK4YOj6aymAjy2-K-NB6BusXJ62tRgATcy1FNG_ziir1qQMUaH9BFpch-X3NfiQlhBqedeFayUq3sIBIS5UL90HB3I7zOKbVioReuS7xx0bg7poSV-G5iVtzKiF6NZTaQE-1Hno2C6bPZgRMsffQ?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/yoEoeNftZPsVNw6uZE8DXkK2ck8oP70LR13FYwdzsvGuK5Bwpd0S0Ez7gu6uHJWiOyC6DblbJn9Vqh4hrA3-wINhhoMLosUwfVEm5S8ErkCz0pPun7vcsy8qB-dsDGTfWiYLWFXldEQqtVVXHXUy2IE5T5XhDTLdB99JPFL1fTNdMPXQ22rvlTcyVWQDcEGR?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/6HH0hH4E-PtRsMWoy39S9a772N1cZN-yDjF6nyYDXv0LclI59MHc9i9SkJWrFSEdTWolLcM7Etk4uvibFL7gq7ryXTGwEZszkAqE1TC5Ei7kWv3Mn0hK2qia5RWyB7ePPa4lusMdgdkCf0eCFEi6ZGuc9-kG95v0IgIME7AfTF-nFUi209qUr4m_cjG4SkAP?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/jgN5Cchz7IoUWpPJyMGVQ28YlvKe1-5giIoe12M2K80xyAHPUCQo6O8F1Wqlyj6m71JOD7xa7ixGZXk2YYfElEVOoaGxWRfIm1DF1QLDxBmS2r3iIdMKL23f40v_tGJ__grXUdtbl-8_2eqvivxTE7rWLm3rtRf1RiLzrTdaDxc_68mvmK-oWxYGkZ9gpXeH?purpose=fullsize)

Docker Hub is simply one example of a registry.

Don't make this conceptual mistake:

> Docker ≠ Docker Hub

They are different things.

### Docker

Docker is the container platform/tooling.

For example:

```bash
docker build
docker run
docker pull
docker push
```

### Docker Hub

Docker Hub is a registry service where images can be stored and retrieved.

```text
Docker CLI
    |
    | push / pull
    v
Docker Hub
```

You can use Docker **without Docker Hub**.

You can also use Docker with another registry.

---

# 9. Public vs private registries

Registries can contain **public** or **private** images.

### Public image

Anyone with access to the registry can pull it.

Example concept:

```text
Registry
   |
   +-- nginx
   +-- redis
   +-- python
```

### Private image

Authentication/authorization is required.

For example, your company may have:

```text
Private Registry

company/
    payment-service
    user-service
    order-service
    frontend
```

Only authorized users or servers can pull them.

This is extremely common in production.

---

# 10. Why companies use private registries

Imagine your company builds:

```text
payment-service
```

You obviously don't want to publish its image publicly.

Instead:

```text
Developer
    |
    | build
    v
payment-service:1.4
    |
    | push
    v
Private Registry
    |
    | pull
    v
Production Kubernetes
```

The registry becomes an important part of your software delivery infrastructure.

---

# 11. Registry vs Repository vs Image

These terms are often confusing.

Let's separate them.

Suppose we have:

```text
registry.example.com/company/payment-service:1.4
```

Conceptually:

```text
registry.example.com
       |
       +-- company/payment-service
                 |
                 +-- 1.4
```

### Registry

The registry is the service/server:

```text
registry.example.com
```

### Repository

The repository is the image collection/name:

```text
company/payment-service
```

### Tag

The tag identifies a particular version/reference:

```text
1.4
```

So:

```text
registry.example.com/company/payment-service:1.4
^^^^^^^^^^^^^^^^^^^^^ ^^^^^^^^^^^^^^^^^^^^^ ^^^
      registry             repository      tag
```

We'll study **image tags properly in Step 8**.

For now, just understand the distinction.

---

# 12. What happens during `docker pull`?

This is an important part of understanding Docker.

Suppose your server executes:

```bash
docker pull registry.example.com/company/payment-service:1.4
```

Docker needs to obtain the image from the registry.

Conceptually:

```text
Server
  |
  | docker pull
  |
  v
Registry
  |
  | image metadata
  | image layers
  v
Docker Engine
  |
  v
Local Image Store
```

Docker downloads the required image content.

After that:

```bash
docker images
```

can show the image locally.

Then:

```bash
docker run registry.example.com/company/payment-service:1.4
```

can create a container from it.

---

# 13. What happens during `docker push`?

Now reverse the direction.

You have:

```text
Local Image
```

and execute:

```bash
docker push registry.example.com/company/payment-service:1.4
```

Conceptually:

```text
Docker Engine
     |
     | image metadata
     | layers
     v
Registry
```

The registry stores the image content.

This means another machine can later retrieve it.

---

# 14. Authentication

Private registries need to know:

> Who are you?

For example:

```bash
docker login registry.example.com
```

Docker authenticates against the registry.

Then:

```bash
docker push registry.example.com/company/payment-service:1.4
```

can be authorized.

Similarly, a production server might authenticate before pulling:

```bash
docker login registry.example.com
```

then:

```bash
docker pull registry.example.com/company/payment-service:1.4
```

In production environments, authentication is usually handled more securely through credentials, service accounts, cloud IAM, Kubernetes image pull secrets, or similar mechanisms.

We don't need to go deep into those yet.

---

# 15. Registry in CI/CD

Now we can connect this concept to CI/CD.

Suppose a developer pushes code:

```text
Developer
    |
    | git push
    v
Git Repository
    |
    v
Jenkins
```

Jenkins builds the Docker image:

```text
Jenkins
   |
   | docker build
   v
Docker Image
   |
   | docker push
   v
Container Registry
```

Then your deployment system retrieves the image:

```text
Container Registry
        |
        | docker pull
        v
Kubernetes
        |
        v
Container
```

So a typical CI/CD flow becomes:

```text
                 CI
                  |
Developer → Git → Jenkins
                  |
                  | build
                  v
             Docker Image
                  |
                  | push
                  v
          Container Registry
                  |
                  | pull
                  v
                 CD
                  |
                  v
             Kubernetes
                  |
                  v
              Container
```

This is why registries are extremely important in CI/CD.

The registry acts as the **bridge between image building and image deployment**.

---

# 16. A very important distinction

Don't think:

> "The registry runs my container."

It doesn't.

A registry **stores and distributes images**.

For example:

```text
Registry
   |
   | stores image
   v
payment-service:1.4
```

Kubernetes or Docker Engine then obtains that image:

```text
Registry
    |
    | pull
    v
Docker/containerd
    |
    | create container
    v
Container
```

So:

```text
Registry → stores/distributes
Docker Engine/containerd → runs
Container → executes application
```

These are different responsibilities.

---

# 17. Beginner misconceptions

## Misconception 1: "Docker Hub is Docker"

No.

```text
Docker       = container tooling/platform
Docker Hub   = image registry service
```

---

## Misconception 2: "A registry builds images"

Usually, no.

The build happens somewhere such as:

```text
Developer machine
Jenkins
GitHub Actions runner
GitLab runner
```

Then the resulting image is pushed to the registry.

```text
Build system
     |
     | build
     v
Image
     |
     | push
     v
Registry
```

---

## Misconception 3: "The registry runs the container"

No.

The registry stores/distributes the image.

A container runtime runs the container.

---

## Misconception 4: "An image only exists in a registry"

No.

An image can exist locally:

```text
Developer machine
    |
    +-- local Docker image
```

and remotely:

```text
Registry
    |
    +-- stored image
```

---

## Misconception 5: "Every push uploads the entire image from scratch"

Not necessarily.

Because Docker images are composed of layers, existing content can be reused.

This is one reason understanding **image layers before registries** was important.

---

# 18. The complete mental model so far

We can now connect all five concepts we've learned.

```text
                    Dockerfile
                        |
                        |
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
                   docker push
                        |
                        v
              +-------------------+
              |  Image Registry   |
              +-------------------+
                        |
                   docker pull
                        |
                        v
                  Docker Image
                        |
                     docker run
                        |
                        v
                    Container
```

This is the foundation you need before moving deeper into CI/CD.

---

# 19. Hands-on: see the concept yourself

You don't need a remote production environment for this lesson.

First check your local images:

```bash
docker images
```

Build a small image:

```bash
docker build -t my-app:1.0 .
```

Check it:

```bash
docker images
```

At this point:

```text
Local Docker Engine
        |
        +-- my-app:1.0
```

If you have a Docker Hub account, you can then tag it with your registry/repository name and push it.

The important thing isn't memorizing these commands yet.

The important sequence is:

```text
docker build
     ↓
local image
     ↓
docker push
     ↓
registry
     ↓
docker pull
     ↓
local image on another machine
     ↓
docker run
```

---

# 20. How this connects to our CI/CD lab

Your planned CI/CD architecture will eventually look roughly like:

```text
                    Git Repository
                          |
                          v
                       Jenkins
                          |
                    docker build
                          |
                          v
                    Docker Image
                          |
                    docker push
                          |
                          v
                 +----------------+
                 | Image Registry |
                 +----------------+
                          |
                     docker pull
                          |
                          v
                    Kubernetes
                          |
                          v
                     Container
```

Later, when we study Kubernetes + Argo CD, this relationship becomes even more important.

For example:

```text
Git
 |
 | application manifests
 v
Argo CD
 |
 | tells Kubernetes what image/version to deploy
 v
Kubernetes
 |
 | pulls image
 v
Registry
```

But **don't worry about that part yet**.

We first need the Docker fundamentals to be solid.

---

# 21. Beginner level vs advanced level

### Beginner — know this now

You should understand:

```text
Image
   ↓ push
Registry
   ↓ pull
Image
   ↓ run
Container
```

And:

* A registry stores/distributes images.
* Docker Hub is one registry.
* Private registries are common in companies.
* `docker push` uploads an image.
* `docker pull` retrieves an image.
* The registry is not the container runtime.
* Images consist of layers, and those layers are part of what gets distributed.

### Advanced — don't worry about yet

Later we can explore:

* Registry API
* OCI image specification
* manifests
* image indexes
* multi-platform images
* content-addressable storage
* digests
* registry authentication
* image signing
* SBOMs
* immutable image references
* garbage collection
* Harbor architecture
* registry mirrors
* Kubernetes image pull secrets

Those are important eventually, but **not necessary for understanding the basic concept**.

---

# 22. The one sentence to remember

If you remember only one thing from this lesson:

> **A Docker image registry is a central place that stores and distributes Docker images so that different machines and environments can obtain the same image.**

And the core flow is:

```text
BUILD → IMAGE → PUSH → REGISTRY → PULL → IMAGE → RUN → CONTAINER
```

That is **Step 5**.

Once this mental model is clear, **Step 6 — Container Filesystem** will explain what happens *inside the container after that image has been pulled and used to create a container*.
