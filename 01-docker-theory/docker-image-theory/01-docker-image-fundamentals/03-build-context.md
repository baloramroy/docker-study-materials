# Phase 1 — Part 3: Build Context

We are continuing exactly from your roadmap:

```text
01. Container vs Image       ✅
02. Dockerfile               ✅
03. Build context            ← NOW
04. Docker image layers
05. Docker image registry
06. Container filesystem
07. Base images
08. Image tags
```

The goal here is **not just to memorize what build context means**. You should understand what Docker actually receives when you run:

```bash
docker build .
```

This is one of the most important concepts for understanding Docker builds and later CI/CD.

---

# 1. What is Build Context?

The **build context** is the set of files and directories that Docker makes available to the build process.

For example:

```text
my-app/
├── Dockerfile
├── app.py
├── requirements.txt
├── config/
│   └── app.conf
└── README.md
```

If you execute:

```bash
docker build .
```

the final `.` means:

> **Use the current directory as the build context.**

So conceptually:

```text
                 docker build .
                       │
                       ▼
              Current directory
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       Dockerfile    app.py    requirements.txt
                       │
                       ▼
                Build Context
```

The important point is:

> **The build context is not the Dockerfile.**

The Dockerfile is the **instructions** for building the image.

The build context is the **set of files available to those instructions**.

---

# 2. Why Does Docker Need a Build Context?

Consider this Dockerfile:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

Docker needs access to:

```text
app.py
```

Where does that file come from?

It comes from the **build context**.

If your directory is:

```text
my-app/
├── Dockerfile
└── app.py
```

and you execute:

```bash
docker build .
```

then:

```text
.  →  my-app/
```

becomes the build context.

Docker can therefore access:

```text
my-app/app.py
```

and the instruction:

```dockerfile
COPY app.py .
```

can work.

---

# 3. The Most Important Relationship

You should remember this relationship:

```text
Dockerfile
    │
    │ instructions
    ▼
Build process
    ▲
    │ files available
    │
Build Context
```

More concretely:

```text
Dockerfile
   │
   ├── FROM
   ├── WORKDIR
   ├── COPY ─────────┐
   ├── RUN           │
   └── CMD           │
                     ▼
               Build Context
                     │
             ┌───────┼────────┐
             ▼       ▼        ▼
           app.py  config/  requirements.txt
```

So when you write:

```dockerfile
COPY app.py /app/
```

Docker looks for `app.py` **inside the build context**.

---

# 4. Understanding `.` in `docker build .`

This command:

```bash
docker build .
```

has two important components:

```text
docker build .
            │
            └── build context
```

The `.` means:

> Current working directory.

Suppose you're here:

```text
/home/user/projects/my-app/
```

and run:

```bash
docker build .
```

then Docker uses:

```text
/home/user/projects/my-app/
```

as the build context.

So:

```text
/home/user/projects/my-app/
├── Dockerfile
├── app.py
├── requirements.txt
└── config/
```

becomes the context.

---

# 5. The Build Context Does NOT Have to Be the Dockerfile Directory

This is very important.

You can specify the Dockerfile separately from the build context.

For example:

```text
project/
├── docker/
│   └── Dockerfile
└── application/
    ├── app.py
    └── requirements.txt
```

You could run:

```bash
docker build -f docker/Dockerfile application/
```

Here:

```text
-f docker/Dockerfile
```

means:

> Use this Dockerfile.

While:

```text
application/
```

means:

> Use `application/` as the build context.

Therefore:

```text
Dockerfile
    │
    │
    ▼
docker/Dockerfile

Build Context
    │
    ▼
application/
├── app.py
└── requirements.txt
```

This distinction is extremely important.

---

# 6. Dockerfile vs Build Context

Let's make the distinction very clear.

| Concept       | Meaning                             |
| ------------- | ----------------------------------- |
| Dockerfile    | Instructions for building the image |
| Build context | Files available to the build        |
| `-f`          | Selects which Dockerfile to use     |
| `.`           | Current directory as build context  |

For example:

```bash
docker build -f docker/Dockerfile application/
```

means:

```text
Dockerfile:
docker/Dockerfile

Build context:
application/
```

---

# 7. Why Can't `COPY` Access Anything on Your Computer?

Suppose your machine looks like this:

```text
/home/user/
├── projects/
│   └── my-app/
│       ├── Dockerfile
│       └── app.py
│
└── secrets/
    └── database-password.txt
```

You build using:

```bash
cd /home/user/projects/my-app

docker build .
```

Your build context is:

```text
/home/user/projects/my-app/
```

Therefore the Dockerfile cannot simply do:

```dockerfile
COPY ../../secrets/database-password.txt /app/
```

Why?

Because:

```text
../../secrets/
```

is outside the build context.

The build is intentionally restricted to the context.

Conceptually:

```text
/home/user/
│
├── projects/
│   └── my-app/       ← Build Context
│       ├── Dockerfile
│       └── app.py
│
└── secrets/          ← Outside context
    └── password.txt
```

Docker's build process cannot use arbitrary files outside that context through normal `COPY`/`ADD` source paths.

---

# 8. Why This Design Exists

This gives Docker a defined boundary.

Without a build context boundary, a Dockerfile could potentially request arbitrary files from the machine.

Instead, you explicitly tell Docker:

> "These are the files I am allowing this build to work with."

For example:

```bash
docker build .
```

means:

```text
Allowed build input
        │
        ▼
   current directory
```

This becomes particularly important in CI/CD.

---

# 9. Build Context and `.dockerignore`

This leads directly to another important concept.

Suppose your project contains:

```text
my-app/
├── Dockerfile
├── app.py
├── requirements.txt
├── .git/
├── node_modules/
├── venv/
├── logs/
└── .env
```

If you run:

```bash
docker build .
```

the build context starts from:

```text
my-app/
```

But you generally don't want unnecessary files to be included in the context.

That's where:

```text
.dockerignore
```

comes in.

For example:

```text
.git/
node_modules/
venv/
logs/
.env
```

Then conceptually:

```text
Project directory
       │
       ▼
.dockerignore
       │
       ▼
Filtered build context
       │
       ├── Dockerfile
       ├── app.py
       └── requirements.txt
```

We'll study `.dockerignore` properly later in the roadmap.

For now, understand:

> **Build context defines what is eligible to be sent to the build, while `.dockerignore` excludes unwanted files from that context.**

---

# 10. A Very Important CI/CD Connection

This is where build context becomes particularly relevant to your learning goal.

Imagine your Jenkins workspace contains:

```text
workspace/
├── application/
├── .git/
├── test-results/
├── logs/
├── credentials/
├── temporary-files/
└── Dockerfile
```

If your pipeline runs:

```bash
docker build .
```

then the context is the workspace directory.

That can be unnecessarily large.

Instead, you might have:

```text
workspace/
├── application/
│   ├── Dockerfile
│   ├── app.py
│   └── requirements.txt
│
├── test-results/
├── logs/
└── temporary-files/
```

and build:

```bash
docker build application/
```

Now:

```text
Build context
      │
      ▼
application/
```

rather than the entire workspace.

This can improve:

* build performance
* network transfer
* CI efficiency
* security
* reproducibility

We'll go much deeper into this later.

---

# 11. What Happens During `docker build`?

At a high level:

```bash
docker build .
```

can be thought of as:

```text
docker build
     │
     ├───────────────┐
     │               │
     ▼               ▼
Dockerfile       Build Context
     │               │
     │               │
     └───────┬───────┘
             ▼
        Build Engine
             │
             ▼
        Image Build
             │
             ▼
        Docker Image
```

The Dockerfile says:

> **What should Docker do?**

The build context says:

> **What files can Docker use while doing it?**

That distinction should become second nature.

---

# 12. Example: Let's Trace a Real Build

Consider:

```text
my-app/
├── Dockerfile
├── app.py
├── requirements.txt
└── config/
    └── app.conf
```

Dockerfile:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY app.py .

COPY config/app.conf ./config/

CMD ["python", "app.py"]
```

Command:

```bash
docker build .
```

The context is:

```text
my-app/
```

Docker can therefore resolve:

```text
requirements.txt
app.py
config/app.conf
```

because they are inside:

```text
my-app/
```

The flow is approximately:

```text
Build context
my-app/
│
├── Dockerfile
├── app.py
├── requirements.txt
└── config/
    └── app.conf
          │
          ▼
      Docker build
          │
          ├── COPY requirements.txt
          │
          ├── RUN pip install
          │
          ├── COPY app.py
          │
          └── COPY config/app.conf
          │
          ▼
       Final image
```

---

# 13. A Common Beginner Mistake

People often think:

> "`docker build` takes the Dockerfile and builds the image."

That's incomplete.

A better mental model is:

> **`docker build` takes a Dockerfile plus a build context and uses them to produce an image.**

Think:

```text
Dockerfile
    +
Build Context
    ↓
Docker Build
    ↓
Image
```

This is the mental model I want you to retain.

---

# 14. Another Important Detail: The Dockerfile Itself

Usually your Dockerfile is inside the build context:

```text
my-app/
├── Dockerfile
├── app.py
└── requirements.txt
```

```bash
docker build .
```

But the Dockerfile can also be elsewhere.

Example:

```text
project/
├── docker/
│   └── production.Dockerfile
└── src/
    ├── app.py
    └── requirements.txt
```

Command:

```bash
docker build \
  -f docker/production.Dockerfile \
  src/
```

Now:

```text
Dockerfile
    ↓
docker/production.Dockerfile

Build context
    ↓
src/
```

This is a very useful pattern in larger repositories.

---

# 15. Build Context Is Not the Final Image

This distinction is also important.

Suppose your context contains:

```text
my-app/
├── Dockerfile
├── app.py
├── README.md
├── tests/
└── docs/
```

That does **not** mean all these files automatically end up in the final image.

The build context is simply the input available to the build.

The Dockerfile determines what gets added to the image.

For example:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

Even if the context contains:

```text
README.md
tests/
docs/
```

they don't automatically become part of the image.

Only:

```text
app.py
```

is explicitly copied by this Dockerfile.

So remember:

```text
Build Context
     │
     │ available to build
     ▼
Dockerfile instructions
     │
     │ selectively use files
     ▼
Image
```

---

# 16. Three Different Things You Must Not Confuse

At this stage, keep these three concepts separate:

### ① Build context

```text
Files available to the build
```

### ② Dockerfile

```text
Instructions telling Docker how to build
```

### ③ Image

```text
The resulting packaged filesystem + configuration
```

So:

```text
                Dockerfile
                    │
                    │ instructions
                    ▼
Build Context ───► Build Process
                       │
                       ▼
                     Image
```

---

# 17. The Command You Should Understand

When you see:

```bash
docker build -t myapp:1.0 .
```

read it mentally as:

```text
docker build
    │
    ├── -t myapp:1.0
    │       │
    │       └── image name/tag
    │
    └── .
        │
        └── build context
```

Don't just memorize the command.

Understand every part.

---

# 18. One More Example

Suppose:

```text
project/
├── Dockerfile
├── src/
│   ├── app.py
│   └── config.py
├── tests/
├── docs/
└── .git/
```

Command:

```bash
docker build .
```

means:

```text
Build context
       │
       ▼
project/
├── Dockerfile
├── src/
├── tests/
├── docs/
└── .git/
```

Then `.dockerignore` can reduce what is sent/considered:

```text
.git/
tests/
docs/
```

resulting conceptually in:

```text
Effective context
       │
       ├── Dockerfile
       └── src/
```

Again, the Dockerfile decides what is actually copied into the image.

---

# 19. What You Should Be Able to Explain Now

Before moving to **Docker image layers**, you should be able to answer these without memorization:

### Question 1

What does this mean?

```bash
docker build .
```

**Answer:**

Build an image using the Dockerfile and the current directory as the build context.

---

### Question 2

What is the build context?

**Answer:**

The set of files and directories made available to the Docker build process as input.

---

### Question 3

Is the Dockerfile the build context?

**Answer:**

No.

```text
Dockerfile = instructions
Build context = available build input
```

---

### Question 4

What does `-f` do?

For example:

```bash
docker build -f docker/Dockerfile app/
```

**Answer:**

`-f` specifies which Dockerfile to use.

`app/` specifies the build context.

---

### Question 5

Does every file in the build context automatically enter the image?

**Answer:**

No.

The Dockerfile determines which files are copied into the image.

---

### Question 6

Why is build context important in CI/CD?

Because an unnecessarily large context can increase:

```text
Build time
   ↓
Data transfer
   ↓
CI resource usage
```

and can expose unnecessary files to the build process.

---

# 20. Your Mental Model

For this phase, I want you to remember exactly this:

```text
                 docker build
                      │
          ┌───────────┴───────────┐
          │                       │
          ▼                       ▼
     Dockerfile              Build Context
          │                       │
          │                       │
   "How to build"          "Files available"
          │                       │
          └───────────┬───────────┘
                      ▼
                 Build Engine
                      │
                      ▼
                  Docker Image
```

And the most important sentence:

> **The Dockerfile defines the build instructions; the build context defines the files available to those instructions.**

That is the core of **Build Context**.

---

## Phase 1 Progress

```text
01. Container vs Image       ✅
02. Dockerfile               ✅
03. Build context            ✅
04. Docker image layers      ← NEXT
05. Docker image registry
06. Container filesystem
07. Base images
08. Image tags
```

The next topic, **Docker image layers**, will build directly on this. We'll connect `FROM`, `COPY`, `RUN`, and the resulting layers, and then you'll see why Dockerfile ordering matters for CI/CD build caching.
