# Phase 1 — Part 2: Dockerfile

We are continuing **strictly in the order of the Docker Image Building Guide**.

```text
Phase 1

01. Container vs Image       ✅
02. Dockerfile               ← NOW
03. Build context
04. Docker image layers
05. Docker image registry
06. Container filesystem
07. Base images
08. Image tags
```

We already understand the first fundamental relationship:

```text
Docker Image
     ↓
creates
     ↓
Container
```

Now the next question is:

> **How do we create a Docker image in a repeatable way?**

That's where the **Dockerfile** comes in.

---

# 1. Basic Concept — What is a Dockerfile?

A **Dockerfile is a text file containing instructions that Docker uses to build an image.**

Think of it as the **build recipe** for an image.

```text
Dockerfile
    │
    │ instructions
    ▼
Docker build process
    │
    ▼
Docker Image
```

For example:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

This file doesn't contain the Docker image itself.

It describes **how the image should be built and how the resulting container should behave**.

So remember:

```text
Dockerfile = instructions
Image      = built artifact
Container  = running instance
```

---

# 2. The Mental Model

A useful mental model is:

```text
                 Dockerfile
              "How to build it"
                     │
                     │ docker build
                     ▼
                Docker Image
                "What we built"
                     │
                     │ docker run
                     ▼
                  Container
                 "What we run"
```

This is the fundamental relationship.

For example:

```bash
docker build -t my-app:1.0 .
```

takes the Dockerfile and produces:

```text
my-app:1.0
```

Then:

```bash
docker run my-app:1.0
```

uses that image to create and start a container.

---

# 3. A Simple Dockerfile

Let's start with a very small example.

Suppose our application is:

```text
my-app/
├── app.py
└── Dockerfile
```

`app.py`:

```python
print("Hello from Docker")
```

Dockerfile:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

At this stage, don't worry about mastering each instruction.

Instead, understand the overall flow:

```text
FROM python:3.12
        ↓
Start from an existing image

WORKDIR /app
        ↓
Set the working directory

COPY app.py .
        ↓
Put application code into the image

CMD ["python", "app.py"]
        ↓
Define the default command when the container starts
```

So the Dockerfile describes the construction and default behavior of the resulting image.

---

# 4. What Docker Actually Does

This is where we want to go deeper than simply saying:

> "Dockerfile is a recipe."

When you run:

```bash
docker build -t my-app:1.0 .
```

Docker reads the Dockerfile and processes its instructions.

Conceptually:

```text
Dockerfile
    │
    ▼
Docker build
    │
    ├── process FROM
    │
    ├── process WORKDIR
    │
    ├── process COPY
    │
    ├── process CMD
    │
    ▼
Resulting image
```

Docker isn't simply storing the Dockerfile inside the image.

Instead, the build process uses the instructions to **construct the image**.

This distinction is important.

```text
Dockerfile
     │
     │ describes
     ▼
Build process
     │
     │ produces
     ▼
Image
```

---

# 5. Dockerfile Instructions Are Not All the Same

One of the most important concepts to understand early is that Dockerfile instructions have different roles.

Consider:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY app.py .

CMD ["python", "app.py"]
```

These instructions don't all mean:

> "Run this command when the container starts."

Instead, some affect the **image build**, while others describe **runtime behavior**.

---

# 6. Build-Time vs Runtime

Consider:

```dockerfile
RUN pip install flask
```

This happens while Docker is **building the image**.

Conceptually:

```text
docker build
     │
     ▼
RUN pip install flask
     │
     ▼
Flask becomes part of image
```

After the image is built, Docker doesn't execute that `RUN` command every time you start a container.

That's a very important distinction.

Now consider:

```dockerfile
CMD ["python", "app.py"]
```

This describes the default command to execute when a container is started.

Conceptually:

```text
docker build
     │
     ▼
Image contains CMD configuration
     │
     │ docker run
     ▼
Container starts
     │
     ▼
python app.py
```

So:

```text
RUN
 │
 └── Build time

CMD
 │
 └── Container runtime
```

We'll study the exact behavior of `RUN`, `CMD`, and `ENTRYPOINT` later.

---

# 7. A Better Classification of Dockerfile Instructions

For now, we can broadly think about the important instructions like this:

| Instruction  | Basic role                                  |
| ------------ | ------------------------------------------- |
| `FROM`       | Select the starting image                   |
| `WORKDIR`    | Set the working directory                   |
| `COPY`       | Copy files into the image                   |
| `RUN`        | Execute something during the build          |
| `ENV`        | Define environment variables                |
| `EXPOSE`     | Declare an intended container port          |
| `CMD`        | Define the default runtime command          |
| `ENTRYPOINT` | Define the main runtime executable/behavior |

Don't memorize this table yet.

We'll study these instructions individually when we reach the dedicated Dockerfile instruction section.

For now, understand that a Dockerfile is made from **different types of instructions that contribute to the final image and/or its runtime behavior**.

---

# 8. What Happens to the Application Files?

Suppose we have:

```text
my-app/
├── app.py
├── requirements.txt
└── Dockerfile
```

And:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY app.py .

CMD ["python", "app.py"]
```

The Dockerfile is telling Docker to construct an image that eventually contains something conceptually like:

```text
/
└── app/
    ├── app.py
    └── requirements.txt
```

along with the environment provided by the starting image and the installed dependencies.

The important point is:

> **The Dockerfile describes how the application's required environment is assembled into the image.**

---

# 9. Dockerfile → Image

Let's follow the complete process.

Project:

```text
my-app/
├── app.py
└── Dockerfile
```

Dockerfile:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

Build:

```bash
docker build -t my-app:1.0 .
```

Conceptually:

```text
             Project
                │
                ├── Dockerfile
                │
                └── app.py
                       │
                       ▼
                 docker build
                       │
                       ▼
                Build process
                       │
                       ▼
                Docker Image
                  my-app:1.0
```

Then:

```bash
docker run my-app:1.0
```

gives:

```text
Docker Image
     │
     │ docker run
     ▼
Container
```

So the entire relationship is:

```text
Dockerfile
    │
    │ docker build
    ▼
Docker Image
    │
    │ docker run
    ▼
Container
```

This is the core mental model for this topic.

---

# 10. Why Not Just Install Everything Manually?

You could technically do something like:

```text
Start container
     ↓
Install Python
     ↓
Copy application
     ↓
Install dependencies
     ↓
Configure application
     ↓
Start application
```

But imagine doing this manually every time.

You could easily forget:

```text
Which package did I install?
Which version?
Which configuration?
Which directory?
Which environment variable?
Which command?
```

And another engineer would have to reproduce the same process.

A Dockerfile turns those steps into a **version-controlled build definition**.

```text
Dockerfile
    │
    ├── repeatable
    ├── reviewable
    ├── version controlled
    └── usable by CI/CD
```

That's one of the major reasons Dockerfiles are so important.

---

# 11. Dockerfile as a Build Definition

Think about a normal application.

You might have:

```text
Source code
    +
Dependencies
    +
Runtime
    +
Configuration
```

The Dockerfile describes how these pieces should be assembled into the image.

Conceptually:

```text
             Dockerfile
                  │
       ┌──────────┼──────────┐
       │          │          │
       ▼          ▼          ▼
    Runtime   Dependencies  App
       │          │          │
       └──────────┼──────────┘
                  ▼
              Image
```

This is why it's reasonable to think of a Dockerfile as part of the application's **build definition**.

---

# 12. Why This Matters for CI/CD

Now connect this to your actual goal.

Imagine your Git repository contains:

```text
my-app/
├── src/
├── requirements.txt
├── Dockerfile
└── ...
```

A developer pushes a change:

```text
Developer
    │
    │ git push
    ▼
Git repository
```

Jenkins checks out the repository:

```text
Git repository
    │
    ▼
Jenkins
```

Then Jenkins can run:

```bash
docker build -t my-app:1.0 .
```

Docker reads the Dockerfile and builds the image.

```text
Git
 │
 │ source + Dockerfile
 ▼
Jenkins
 │
 │ docker build
 ▼
Docker Image
 │
 │ docker push
 ▼
Registry
```

This is why a Dockerfile fits naturally into CI/CD.

The CI server doesn't need a long list of manually remembered commands for building the application's environment.

The build definition is stored in the repository.

---

# 13. Dockerfile and Reproducibility

Suppose you build an image today.

Then tomorrow you need to build it again.

If the build process is represented by a Dockerfile:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

you have a documented build process.

The idea is:

```text
Same Dockerfile
      +
Same build inputs
      ↓
Same intended image build
```

There are additional factors affecting strict reproducibility, such as mutable dependencies and base-image references, but those are advanced topics.

For now, the important idea is:

> **A Dockerfile makes the image-building process explicit and repeatable instead of relying on someone's memory.**

---

# 14. Dockerfile Is Not a Shell Script

This is an important beginner misconception.

A Dockerfile can contain commands that look like shell commands:

```dockerfile
RUN apt-get update
RUN pip install flask
```

But a Dockerfile itself is **not simply a Bash script**.

For example:

```dockerfile
WORKDIR /app
```

is a Dockerfile instruction.

It's not a shell command.

Similarly:

```dockerfile
COPY app.py /app/
```

is a Dockerfile instruction.

Docker interprets it as part of the image-building process.

So:

```text
Dockerfile
    ≠
Bash script
```

It is its own instruction language understood by Docker's build system.

---

# 15. Beginner Misconception: Dockerfile = Image

No.

These are different things.

### Dockerfile

```text
Text file
```

Example:

```dockerfile
FROM python:3.12
COPY app.py /app/
```

### Image

```text
Built artifact
```

Example:

```text
my-app:1.0
```

### Container

```text
Running/created instance of the image
```

Example:

```text
my-app-container
```

The relationship is:

```text
Dockerfile
    │
    │ build
    ▼
Image
    │
    │ run
    ▼
Container
```

---

# 16. Beginner Misconception: `RUN` Happens When the Container Starts

No.

Consider:

```dockerfile
RUN pip install flask
```

That happens during:

```bash
docker build
```

not during:

```bash
docker run
```

Conceptually:

```text
docker build
     │
     ▼
RUN pip install flask
     │
     ▼
Flask included in image
     │
     │
     │ later
     ▼
docker run
     │
     ▼
Container starts
```

This distinction will become extremely important when we study image layers.

---

# 17. Beginner Misconception: `CMD` Builds the Image

Not exactly.

Consider:

```dockerfile
CMD ["python", "app.py"]
```

The `CMD` instruction doesn't execute the application during the image build.

Instead, it specifies the **default command for the container**.

Conceptually:

```text
docker build
     │
     ▼
Image stores CMD configuration
     │
     ▼
docker run
     │
     ▼
Default command starts
```

So:

```text
RUN  → build-time execution
CMD  → runtime default
```

---

# 18. Beginner Misconception: Dockerfile Instructions Are Executed Once Forever

Be careful with this mental model.

A Dockerfile is used to **build an image**.

The resulting image is then used to create containers.

So the relationship is more like:

```text
Dockerfile
    │
    │ build
    ▼
Image
    │
    ├── Container 1
    ├── Container 2
    └── Container 3
```

You don't need to rebuild the image just because you want another container from it.

You can create multiple containers from the same image.

---

# 19. One Dockerfile Can Produce Different Image Versions

Suppose your Dockerfile is:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY . .

CMD ["python", "app.py"]
```

You can build:

```bash
docker build -t my-app:1.0 .
```

Then later:

```bash
docker build -t my-app:1.1 .
```

The Dockerfile can be the same while the build inputs or source code have changed.

Conceptually:

```text
Dockerfile + Build inputs
          │
          ▼
       Image 1.0

Later:

Dockerfile + Updated inputs
          │
          ▼
       Image 1.1
```

We'll learn more about the build inputs in **Step 3: Build Context**.

---

# 20. A Small Hands-On Example

Let's create the smallest useful example.

Directory:

```text
dockerfile-demo/
├── Dockerfile
└── app.py
```

`app.py`:

```python
print("Hello from Docker")
```

Dockerfile:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

Build it:

```bash
docker build -t dockerfile-demo:1.0 .
```

Then run:

```bash
docker run --rm dockerfile-demo:1.0
```

You should get:

```text
Hello from Docker
```

Now let's understand what happened.

---

# 21. What Happened During `docker build`?

You ran:

```bash
docker build -t dockerfile-demo:1.0 .
```

Docker read:

```dockerfile
FROM python:3.12
```

and used the specified starting image.

Then:

```dockerfile
WORKDIR /app
```

established the working directory.

Then:

```dockerfile
COPY app.py .
```

put the application file into the image.

Then:

```dockerfile
CMD ["python", "app.py"]
```

specified the default runtime command.

The result was:

```text
dockerfile-demo:1.0
```

---

# 22. What Happened During `docker run`?

You then executed:

```bash
docker run --rm dockerfile-demo:1.0
```

Docker used the image to create a container.

The image contained the `CMD` configuration:

```text
python app.py
```

So the container started with that command.

Conceptually:

```text
Image
  │
  │ docker run
  ▼
Container
  │
  ▼
python app.py
  │
  ▼
Hello from Docker
```

Notice something important:

**The Dockerfile itself was not running inside the container.**

The Dockerfile was used earlier to build the image.

---

# 23. Dockerfile and Image Layers — Preview Only

We are intentionally **not studying image layers yet**.

But there is one relationship worth knowing.

When Docker processes instructions such as:

```dockerfile
COPY app.py .
RUN pip install flask
```

the resulting image is constructed from filesystem changes and metadata produced during the build.

This is one reason Dockerfiles and image layers are closely connected.

For now, simply remember:

```text
Dockerfile instructions
        ↓
Build operations
        ↓
Image
```

In **Step 4**, we'll open this up properly and understand how the resulting image is actually composed of layers.

---

# 24. What About `FROM`?

You will see this in almost every Dockerfile:

```dockerfile
FROM python:3.12
```

For now, understand only this:

> `FROM` specifies the starting image for the build.

Conceptually:

```text
Existing image
      │
      ▼
Dockerfile build
      │
      ▼
Your image
```

We will study **base images** properly in Step 7.

So don't worry yet about:

* where base images come from
* how base images are constructed
* minimal images
* `scratch`
* Alpine
* Debian
* distroless

Those belong later.

---

# 25. What About the `.` in `docker build`?

You will repeatedly see:

```bash
docker build -t my-app:1.0 .
```

There is a `.` at the end.

For now, **don't try to memorize what it means**.

It is related to the files Docker makes available to the build.

That's exactly what we'll study next:

> **Step 3 — Build Context**

So for now:

```text
docker build -t my-app:1.0 .
                              ↑
                         later topic
```

This deliberate separation is important.

We don't want to mix the concepts prematurely.

---

# 26. Core Dockerfile Instructions — Initial Map

At this stage, you should have a basic map of the important instructions:

```text
FROM
  ↓
Starting point

WORKDIR
  ↓
Working directory

COPY
  ↓
Bring files into image

RUN
  ↓
Perform build-time operation

ENV
  ↓
Environment configuration

EXPOSE
  ↓
Declare intended port

CMD
  ↓
Default runtime command

ENTRYPOINT
  ↓
Main runtime executable/behavior
```

Later, we'll study them **one by one**, including:

* syntax
* actual behavior
* common mistakes
* best practices
* interaction with other instructions
* CI/CD implications

Don't memorize them yet.

---

# 27. Dockerfile in a Real Project

A realistic application might look like:

```text
payment-service/
├── src/
├── tests/
├── requirements.txt
├── Dockerfile
├── .gitignore
└── README.md
```

The Dockerfile becomes part of the project itself.

Git tracks it:

```text
Git repository
     │
     ├── application source
     ├── tests
     ├── requirements.txt
     └── Dockerfile
```

Now every developer and CI server can use the same build definition.

That is a major improvement over:

```text
"Ask Rahim how he built the production image."
```

Instead:

```text
"Read the Dockerfile."
```

That's the engineering value of treating infrastructure/build configuration as code.

---

# 28. Why Dockerfile Matters So Much in CI/CD

Now connect everything to your eventual pipeline.

A simplified pipeline might be:

```text
Developer
    │
    │ git push
    ▼
Git Repository
    │
    │ source + Dockerfile
    ▼
Jenkins
    │
    │ docker build
    ▼
Docker Image
    │
    │ docker push
    ▼
Registry
    │
    ▼
Deployment
```

The Dockerfile is therefore sitting right in the middle of the **application-to-image** part of the CI pipeline.

Later, you'll see something like:

```bash
docker build \
  -t registry.example.com/my-app:${BUILD_NUMBER} .
```

The CI system supplies the build command.

But the Dockerfile defines **how the application becomes an image**.

---

# 29. A Crucial Mental Model for CI/CD

Think of these as three separate things:

```text
Source Code
    │
    │ what developers write
    ▼
Dockerfile
    │
    │ how application becomes image
    ▼
Docker Image
    │
    │ deployable artifact
    ▼
Container
```

This distinction becomes very important later.

For example:

```text
Git commit
    ↓
CI build
    ↓
Docker image
    ↓
Registry
    ↓
Kubernetes
    ↓
Container
```

The Dockerfile is the **bridge between application source and the container image**.

---

# 30. Beginner Concepts vs Advanced Concepts

At this point, you should understand:

### Beginner concepts

* What a Dockerfile is
* Why it exists
* `Dockerfile → docker build → Image`
* `Image → docker run → Container`
* Dockerfile instructions
* Build-time vs runtime distinction
* Basic purpose of `FROM`, `WORKDIR`, `COPY`, `RUN`, `ENV`, `EXPOSE`, `CMD`, `ENTRYPOINT`
* Why Dockerfiles are useful for reproducible builds
* Why Dockerfiles belong in source control
* Why Dockerfiles are important to CI/CD

### Don't worry about these yet

We will deliberately postpone:

* Build context
* `.dockerignore`
* Image layers
* Build cache
* Multi-stage builds
* BuildKit
* Base-image internals
* Image registries
* Tags and digests
* Image optimization
* Security scanning
* SBOM
* Image signing
* Advanced `ENTRYPOINT`/`CMD` behavior

Those concepts have their own places in the roadmap.

---

# 31. The One-Sentence Takeaway

If you remember only one thing from this entire topic:

> **A Dockerfile is a version-controlled set of instructions that Docker's build system uses to construct a Docker image and define important aspects of how containers created from that image should behave.**

The core relationship is:

```text
                 Dockerfile
              "How to build"
                     │
                     │ docker build
                     ▼
                 Docker Image
               "What was built"
                     │
                     │ docker run
                     ▼
                  Container
                 "What runs"
```

And this is the foundation we'll build on throughout the rest of Phase 1.

---

# Phase 1 Progress

```text
01. Container vs Image       ✅
02. Dockerfile               ✅
03. Build context            ← NEXT
04. Docker image layers
05. Docker image registry
06. Container filesystem
07. Base images
08. Image tags
```

The **next topic is Build Context**.

And this time, when we see:

```bash
docker build -t my-app:1.0 .
```

we'll finally answer the question we intentionally postponed:

> **What exactly does that `.` mean, what files does Docker actually send/use during the build, and why does Docker need a build context at all?**
