# Phase 1 — Part 3: Build Context

## Where we are

```text
Phase 1 — Docker Fundamentals

01. Container vs Image       ✅
02. Dockerfile               ✅
03. Build Context             ← NOW
04. Docker Image Layers
05. Docker Image Registry
06. Container Filesystem
07. Base Images
08. Image Tags
```

## 1. What is Build Context?

The **build context** is the set of files that Docker makes available to the image-building process.

Consider:

```text
my-app/
├── Dockerfile
├── app.py
├── requirements.txt
└── config/
    └── app.conf
```

If you run:

```bash
docker build -t my-app:1.0 .
```

the final `.` means:

> Use the current `my-app/` directory as the build context.

So:

```text
my-app/
├── Dockerfile
├── app.py
├── requirements.txt
└── config/
    └── app.conf
         │
         │ Build Context
         ▼
    Docker Build
```

---

# 2. Why Does Docker Need a Build Context?

Because the Dockerfile may need files from your project.

For example:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY app.py .
COPY requirements.txt .
```

Docker needs access to:

```text
app.py
requirements.txt
```

The build context provides those files.

The two concepts therefore have different jobs:

```text
Dockerfile
    │
    │ tells Docker what to do
    │
    ▼
Docker Build
    ▲
    │
    │ provides available files
Build Context
```

A useful way to remember it:

> **Dockerfile = instructions**
> **Build context = available input files**

---

# 3. Build Context Does Not Mean Image Contents

This is one of the most important points.

Suppose your context is:

```text
my-app/
├── Dockerfile
├── app.py
├── requirements.txt
├── README.md
└── tests/
```

The entire directory is the **build context**.

But your image might contain only:

```text
/app/
├── app.py
└── requirements.txt
```

if the Dockerfile says:

```dockerfile
COPY app.py .
COPY requirements.txt .
```

So:

```text
Build Context
     │
     ├── app.py            ──→ copied into image
     ├── requirements.txt  ──→ copied into image
     ├── README.md         ──→ not copied
     └── tests/            ──→ not copied
```

Therefore:

> **A file being inside the build context does not automatically put that file into the image.**

The Dockerfile determines what gets used.

---

# 4. What Does the `.` Mean?

We've already seen:

```bash
docker build -t my-app:1.0 .
```

The last argument specifies the build context.

So:

```text
docker build -t my-app:1.0 .
                              ↑
                       build context
```

If you're currently in:

```text
/home/user/my-app
```

then:

```bash
docker build -t my-app:1.0 .
```

means:

```text
Build context =
/home/user/my-app
```

You can also explicitly specify another directory:

```bash
docker build -t my-app:1.0 /home/user/my-app
```

The context is still:

```text
/home/user/my-app
```

---

# 5. The Build Context Boundary

Docker builds cannot simply access arbitrary files from your host filesystem.

Suppose:

```text
/home/user/
├── secret.txt
│
└── my-app/
    ├── Dockerfile
    └── app.py
```

You run:

```bash
cd /home/user/my-app
docker build -t my-app:1.0 .
```

The context is:

```text
/home/user/my-app
```

Therefore this is outside the context:

```text
/home/user/secret.txt
```

You cannot simply do:

```dockerfile
COPY ../secret.txt /app/
```

to access it.

This boundary is important for both **correctness and security**.

---

# 6. Dockerfile and Build Context Are Separate

The Dockerfile and context don't have to be the same location.

For example:

```bash
docker build \
  -f /home/user/my-app/Dockerfile \
  -t my-app:1.0 \
  /home/user/my-app
```

Here:

```text
-f /home/user/my-app/Dockerfile
       ↓
Dockerfile

/home/user/my-app
       ↓
Build context
```

This becomes useful when a project has multiple Dockerfiles:

```text
my-app/
├── Dockerfile
├── Dockerfile.dev
├── Dockerfile.test
└── src/
```

For example:

```bash
docker build -f Dockerfile.dev -t my-app:dev .
```

The important concept is simply:

> **Dockerfile location and build-context location are separate concepts.**

---

# 7. `.dockerignore`

The build context can contain files that shouldn't be sent to the builder.

For example:

```text
my-app/
├── Dockerfile
├── app.py
├── .git/
├── node_modules/
├── logs/
├── .env
└── README.md
```

You normally don't want unnecessary files such as `.git`, logs, or local dependencies involved in the build context.

Docker provides:

```text
.dockerignore
```

For example:

```text
.git
node_modules
logs
.env
```

Conceptually:

```text
Project Directory
       │
       │ .dockerignore
       ▼
Filtered Build Context
       │
       ▼
Docker Build
```

We'll study `.dockerignore` more deeply later when we discuss image/build optimization.

For now, remember:

> **Build context defines the available input, while `.dockerignore` can exclude files from that input.**

---

# 8. Hands-on Exercise

Let's verify the concepts.

Create:

```text
docker-context-demo/
├── Dockerfile
├── app.txt
└── secret.txt
```

Put some text into both files.

Dockerfile:

```dockerfile
FROM alpine

WORKDIR /app

COPY app.txt .
```

Build:

```bash
docker build -t context-demo:1.0 .
```

Run:

```bash
docker run --rm context-demo:1.0 ls -l /app
```

You should see:

```text
app.txt
```

but not:

```text
secret.txt
```

Why?

Both files were inside the **build context**, but only `app.txt` was instructed to be copied into the image.

That's the distinction:

```text
Build Context
│
├── app.txt       ── COPY ──→ Image
│
└── secret.txt    ── not copied
```

---

# 9. Failure Case — Outside the Context

Now create:

```text
/home/user/
├── outside.txt
└── docker-context-demo/
    ├── Dockerfile
    └── app.txt
```

Use this Dockerfile:

```dockerfile
FROM alpine

COPY ../outside.txt /app/
```

Then:

```bash
cd docker-context-demo
docker build -t context-demo:2.0 .
```

The build should fail because:

```text
../outside.txt
```

is outside the build context.

The important lesson is:

```text
Build Context
┌─────────────────────────┐
│ Dockerfile              │
│ app.txt                 │
└─────────────────────────┘
          │
          │ Docker build
          ▼
Files inside context
are available

outside.txt
     ↑
outside the boundary
```

---

# 10. Why Build Context Matters in CI/CD

Suppose Jenkins checks out your application:

```text
jenkins-workspace/
└── my-app/
    ├── Dockerfile
    ├── src/
    ├── tests/
    └── README.md
```

The pipeline runs:

```bash
docker build -t my-app:${BUILD_NUMBER} .
```

If the working directory is `my-app/`, then that directory becomes the build context.

Therefore, the build context determines:

* Which files are available to the build
* What Docker can access with `COPY`/`ADD`
* How much data is provided to the builder
* What unnecessary files should potentially be excluded

This becomes particularly important when we optimize Docker builds for CI/CD.

---

# 11. The Mental Model

At this point, keep this model:

```text
                    docker build
                         │
          ┌──────────────┴──────────────┐
          │                             │
     Dockerfile                  Build Context
     instructions                     files
          │                             │
          └──────────────┬──────────────┘
                         ▼
                    Image Build
                         │
                         ▼
                    Docker Image
```

In one sentence:

> **The Dockerfile tells Docker what to do, while the build context determines which files are available to do it with.**

---

# 12. What You Should Know Now

You should be able to answer:

**What does `.` mean?**

> The current directory is being used as the build context.

**Does every file in the context become part of the image?**

> No. The Dockerfile determines which files are used/copied.

**Can `COPY` normally access files outside the build context?**

> No.

**Can the Dockerfile and build context be specified separately?**

> Yes.

**What does `.dockerignore` do?**

> It excludes specified files/directories from the build context sent to the builder.

---

# Phase 1 Progress

```text
Phase 1 — Docker Fundamentals

01. Container vs Image       ✅
02. Dockerfile               ✅
03. Build Context             ✅
04. Docker Image Layers       ← NEXT
05. Docker Image Registry
06. Container Filesystem
07. Base Images
08. Image Tags
```

We've now established the three foundations:

```text
Dockerfile
    │
    │ instructions
    ▼
Docker Build ◄──── Build Context
    │                    │
    │                    └── available files
    ▼
Docker Image
```

**Next: Phase 1 — Part 4: Docker Image Layers.**
