# Phase 2 — Build Your First Docker Image

## Part 1 — What We're Building & Why

### Where We Are

Phase 1 established the Docker fundamentals we need:

```text
Phase 1
├── Container vs Image
├── Dockerfile
├── Build Context
├── Image Layers
├── Registry
├── Container Filesystem
├── Base Images
└── Image Tags
```

We won't re-teach those concepts here.

Instead, we'll use them as we start doing something practical:

> **Take a simple application and turn it into a Docker image.**

---

# 1. Our First Application

We're going to use a very small Python application.

Create this directory:

```text
my-app/
```

Inside it, create:

```text
my-app/
└── app.py
```

Put this in `app.py`:

```python
print("Hello from Docker!")
```

That's the entire application.

We aren't trying to learn Python here. Python is simply giving us something concrete to package into a Docker image.

If Python is installed directly on a machine, we could run:

```bash
python3 app.py
```

and get:

```text
Hello from Docker!
```

Our goal is to make Docker run the same application.

---

# 2. What Are We Trying to Produce?

We currently have:

```text
my-app/
└── app.py
```

We want to turn that application into an image:

```text
app.py
   │
   │ Docker build
   ▼
Docker Image
my-app:1.0
```

Then that image can be used to create a container:

```text
Docker Image
my-app:1.0
     │
     │ docker run
     ▼
Container
     │
     ▼
python app.py
```

So the basic journey is:

```text
Application
    ↓
Docker Image
    ↓
Container
    ↓
Application runs
```

This is the practical workflow we'll build throughout Phase 2.

---

# 3. Why Do We Need an Image?

Our application depends on an environment in which it can run.

For our tiny example, that means at least:

```text
Python
+
Application
```

For a real application, the environment could be much larger:

```text
Application
    │
    ├── Runtime
    ├── Libraries
    ├── Dependencies
    ├── OS components
    ├── Configuration
    └── Application files
```

We want the application and its required environment to be packaged consistently.

That's what our Docker image will provide.

Conceptually:

```text
Docker Image
├── Runtime
├── Required environment
└── Application
```

Then:

```text
Image
  │
  ▼
Container
  │
  ▼
Application
```

---

# 4. Where Does the Dockerfile Come In?

We need to tell Docker **how to construct the image**.

That's the job of the Dockerfile.

Our project will therefore become:

```text
my-app/
├── app.py
└── Dockerfile
```

The Dockerfile will contain instructions such as:

```text
Start from an appropriate image
        ↓
Set up the application directory
        ↓
Copy the application
        ↓
Install dependencies if necessary
        ↓
Define what should run
```

Docker reads those instructions during the image build.

The basic relationship is:

```text
                 Dockerfile
                     │
              build instructions
                     │
                     ▼
Application ───► docker build
                     │
                     ▼
                Docker Image
```

---

# 5. Dockerfile as a Recipe

A useful mental model is:

> **A Dockerfile is a recipe for constructing an image.**

Suppose someone else gets our project:

```text
my-app/
├── app.py
└── Dockerfile
```

They shouldn't need to know every manual step we performed to prepare the environment.

The Dockerfile records those steps.

That gives us a repeatable process:

```text
Same application
       +
Same Dockerfile
       │
       ▼
docker build
       │
       ▼
Docker image
```

This repeatability becomes especially important later when the image is built automatically in CI/CD.

For now, just remember:

> **The Dockerfile describes how the image should be built.**

---

# 6. A Phase 1 Callback: Build Context

We already covered **build context** in Phase 1, so we won't re-teach it here.

For this project, we'll use:

```text
my-app/
├── app.py
└── Dockerfile
```

and eventually run:

```bash
docker build -t my-app:1.0 .
```

The `.` tells Docker:

> Use the current directory as the build context.

That's important because later we'll use Dockerfile instructions such as `COPY` to bring files from that build context into the image.

That's all we need to recall here.

---

# 7. The Complete Build Picture

We now have all the pieces:

```text
                    PROJECT
                      │
          ┌───────────┴───────────┐
          │                       │
       app.py                 Dockerfile
          │                       │
          │                  build instructions
          │                       │
          └───────────┬───────────┘
                      │
                Build Context
                      │
                      │ docker build
                      ▼
                 Docker Image
                  my-app:1.0
                      │
                      │ docker run
                      ▼
                   Container
                      │
                      ▼
                  app.py runs
```

Notice that we haven't actually written the Dockerfile yet.

That's intentional.

Before doing that, we need to understand what the individual Dockerfile instructions actually mean.

That's the next part.

---

# Part 1 Checkpoint

At this point, you should be able to explain these four things:

### What is our application?

```text
app.py
```

### What is the Dockerfile?

A set of instructions describing **how to build the image**.

### What does `docker build` produce?

```text
Docker Image
```

For our example:

```text
my-app:1.0
```

### What happens after we have the image?

The image can be used to create a container:

```text
Image
  ↓
Container
  ↓
Application
```

---

## The Mental Model

Keep this simple picture in mind:

```text
        APPLICATION
           app.py
              │
              │
              ▼
         Dockerfile
      build instructions
              │
              │ docker build
              ▼
        DOCKER IMAGE
          my-app:1.0
              │
              │ docker run
              ▼
          CONTAINER
              │
              ▼
       APPLICATION RUNS
```

**Part 1 is complete.**

We have established **what we're building and why** without repeating the Phase 1 material.

In **Part 2**, we'll finally open the Dockerfile and learn the first three instructions:

```dockerfile
FROM
WORKDIR
COPY
```

These three will answer a very practical question:

> **Where does our image start, where do we work inside it, and how does our application get into it?**
