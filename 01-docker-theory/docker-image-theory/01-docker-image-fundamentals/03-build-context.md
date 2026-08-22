# Phase 1 — Part 3: Docker Build Context

We are continuing **strictly in the order of the Docker Image Building Guide**.

```text
Phase 1

01. Container vs Image       ✅
02. Dockerfile               ✅
03. Build context            ← NOW
04. Docker image layers
05. Docker image registry
06. Container filesystem
07. Base images
08. Image tags
```

In Step 2, we deliberately postponed explaining this command:

```bash
docker build -t my-app:1.0 .
```

Today, we'll answer the important question:

> **What exactly does the `.` mean, what files are available to Docker during the build, and why does Docker need a build context?**

---

# 1. Basic Concept — What Is Build Context?

The **build context** is the set of files and directories that Docker makes available to the build process.

The simplest example is:

```bash
docker build -t my-app:1.0 .
```

The final:

```text
.
```

specifies the **build context**.

In this case:

> `.` means "use the current directory as the build context."

So if you're currently inside:

```text
my-app/
```

then:

```bash
docker build -t my-app:1.0 .
```

means:

```text
Current directory
       │
       │ build context
       ▼
Docker build
```

---

# 2. The Mental Model

Think of the build context as a **package of files that the Docker build is allowed to access**.

For example:

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

then:

```text
                 Current directory
                        │
                        │
                        ▼
              ┌──────────────────┐
              │   Build Context  │
              │                  │
              │ Dockerfile       │
              │ app.py           │
              │ requirements.txt │
              │ config/          │
              └──────────────────┘
                        │
                        ▼
                  Docker Build
```

The important idea is:

> **The build context defines the filesystem area available to the build.**

---

# 3. Why Does Docker Need a Build Context?

Consider this Dockerfile:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY app.py .
```

Docker needs to find:

```text
app.py
```

Where does it come from?

It comes from the **build context**.

So:

```text
Build context
     │
     │ contains
     ▼
  app.py
     │
     │ COPY
     ▼
Docker image
```

The Dockerfile tells Docker:

```dockerfile
COPY app.py .
```

But the build context provides the actual:

```text
app.py
```

---

# 4. Dockerfile and Build Context Are Different

This distinction is extremely important.

Suppose:

```text
my-app/
├── Dockerfile
├── app.py
└── requirements.txt
```

There are two separate concepts:

### Dockerfile

Tells Docker:

> **What should I do?**

### Build context

Provides:

> **Which files are available to the build?**

So:

```text
Dockerfile
   │
   │ instructions
   ▼
Build process
   ▲
   │ files
   │
Build context
```

This distinction becomes very important when we reach `COPY`.

---

# 5. Understanding the `.`

Now let's finally explain:

```bash
docker build -t my-app:1.0 .
```

Break it apart:

```text
docker build
```

→ Start an image build.

```text
-t my-app:1.0
```

→ Give the resulting image a name/tag.

```text
.
```

→ Use the current directory as the build context.

So:

```text
docker build -t my-app:1.0 .
                         ↑
                         │
                    build context
```

This is why the `.` matters.

---

# 6. Build Context Does Not Mean "Dockerfile Location"

A common beginner misconception is:

> "The build context is the directory where the Dockerfile is."

Often that's true, but **it doesn't have to be**.

The Dockerfile and build context are separate concepts.

For example:

```text
project/
├── Dockerfile
├── src/
│   └── app.py
└── config/
```

You can use:

```bash
docker build -t my-app:1.0 .
```

Here both are in the same directory.

But Docker also allows you to specify a different Dockerfile:

```bash
docker build -f docker/Dockerfile -t my-app:1.0 .
```

Conceptually:

```text
Dockerfile
    │
    │ explicitly selected with -f
    ▼
docker/Dockerfile

Build context
    │
    │ .
    ▼
project/
```

So:

> **Dockerfile location and build-context location are not inherently the same thing.**

---

# 7. The Most Important Rule: `COPY` Works Within the Build Context

Suppose:

```text
project/
├── Dockerfile
├── app.py
└── config.txt
```

Dockerfile:

```dockerfile
FROM python:3.12

COPY app.py /app/
COPY config.txt /app/
```

If you run:

```bash
docker build -t my-app:1.0 .
```

then Docker can access:

```text
app.py
config.txt
```

because they're inside the build context.

Conceptually:

```text
Build Context
├── Dockerfile
├── app.py        ──────┐
└── config.txt    ──────┤
                         │
                         ▼
                    Docker build
                         │
                         ▼
                       COPY
```

---

# 8. What If the File Is Outside the Context?

Suppose your filesystem is:

```text
workspace/
├── shared/
│   └── common.conf
│
└── my-app/
    ├── Dockerfile
    └── app.py
```

You run:

```bash
cd workspace/my-app
```

and:

```bash
docker build -t my-app:1.0 .
```

Your build context is:

```text
my-app/
```

It does **not** include:

```text
workspace/shared/
```

Therefore this:

```dockerfile
COPY ../shared/common.conf /app/
```

is not a normal way to escape the build context.

The important mental model is:

```text
Build Context
┌──────────────────────┐
│ Dockerfile           │
│ app.py               │
│ requirements.txt     │
│                      │
│                      │
└──────────────────────┘
          │
          │ allowed source
          ▼
       Docker build
```

Outside the boundary:

```text
shared/common.conf
        X
```

is not part of that context.

---

# 9. Why Does Docker Have This Boundary?

At first, this might seem inconvenient.

But the boundary is intentional.

Imagine your project directory contains:

```text
my-app/
├── Dockerfile
├── app.py
├── requirements.txt
├── .git/
├── secrets/
├── backups/
└── database-dump.sql
```

If Docker automatically had access to your entire machine, the build would have access to far more data than it needs.

Instead, you explicitly choose:

```bash
docker build ... <context>
```

The context defines what is available.

So conceptually:

```text
Your computer
┌───────────────────────────────────────┐
│                                       │
│  Other files                          │
│                                       │
│       ┌─────────────────────┐         │
│       │ Build Context       │         │
│       │                     │         │
│       │ Files available     │         │
│       │ to this build       │         │
│       └─────────────────────┘         │
│                                       │
└───────────────────────────────────────┘
```

This is a deliberate boundary.

---

# 10. Build Context Is Not the Final Image

Another important distinction.

Suppose:

```text
my-app/
├── Dockerfile
├── app.py
├── test.py
└── README.md
```

All of these may be part of the build context.

But that does **not** mean all of them automatically become part of the final image.

For example:

```dockerfile
FROM python:3.12

COPY app.py /app/
```

The resulting image might contain:

```text
/app/app.py
```

but not:

```text
/app/test.py
/app/README.md
```

So:

```text
Build Context
    │
    │ provides available files
    ▼
Docker Build
    │
    │ instructions decide what is used
    ▼
Docker Image
```

This is a very important mental model.

---

# 11. Build Context vs `COPY`

Let's make this even clearer.

Suppose:

```text
my-app/
├── Dockerfile
├── app.py
├── test.py
└── README.md
```

Build context:

```text
my-app/
```

Dockerfile:

```dockerfile
FROM python:3.12

COPY app.py /app/
```

Then:

```text
Build Context
│
├── app.py ──────────────► copied into image
│
├── test.py
│
└── README.md
```

Only `app.py` is selected by the `COPY` instruction.

Therefore:

> **Being inside the build context means a file is available to the build; it does not mean Docker automatically copies it into the image.**

---

# 12. The Build Context Is the Build's Input

A useful way to think about it is:

```text
Build Context
       +
Dockerfile
       │
       ▼
   Build Process
       │
       ▼
   Docker Image
```

The Dockerfile describes the operations.

The build context provides the files that those operations may need.

So:

```text
Dockerfile
= instructions

Build context
= available build input
```

Together they allow Docker to construct the image.

---

# 13. What About the Dockerfile Itself?

Usually the Dockerfile is inside the build context.

For example:

```text
my-app/
├── Dockerfile
└── app.py
```

with:

```bash
docker build -t my-app:1.0 .
```

Here:

```text
. → my-app/
```

and the Dockerfile is inside that context.

Docker automatically looks for a file named:

```text
Dockerfile
```

by default.

You can also specify another file:

```bash
docker build -f Dockerfile.prod -t my-app:prod .
```

Now:

```text
Dockerfile.prod
```

is explicitly selected as the Dockerfile.

But the context remains:

```text
.
```

So again:

```text
Dockerfile
     ≠
Build context
```

They are related, but separate.

---

# 14. The `-f` Option Makes This Clear

Suppose:

```text
project/
├── docker/
│   ├── Dockerfile.dev
│   └── Dockerfile.prod
│
├── src/
│   └── app.py
│
└── requirements.txt
```

You can run:

```bash
docker build \
    -f docker/Dockerfile.prod \
    -t my-app:prod \
    .
```

Now there are two separate paths:

```text
-f docker/Dockerfile.prod
        │
        └── Dockerfile

.
│
└── Build context
```

The build context is still the entire:

```text
project/
```

directory.

This distinction becomes useful in larger repositories.

---

# 15. Build Context and Large Projects

Imagine a large project:

```text
project/
├── backend/
├── frontend/
├── documentation/
├── tests/
├── videos/
├── backups/
├── node_modules/
├── .git/
└── Dockerfile
```

If you run:

```bash
docker build -t my-app:1.0 .
```

your context is:

```text
project/
```

That's potentially a lot of data.

But perhaps the Docker image only needs:

```text
backend/
Dockerfile
requirements.txt
```

Sending a huge amount of unnecessary data as build context can make builds less efficient.

This leads directly to an important Docker feature:

```text
.dockerignore
```

---

# 16. `.dockerignore`

A `.dockerignore` file tells Docker which files should be excluded from the build context sent/used for the build.

For example:

```text
node_modules
.git
*.log
.env
tests/
```

Conceptually:

```text
Project
│
├── app.py                ✓ context
├── Dockerfile            ✓ context
├── requirements.txt      ✓ context
│
├── .git/                 ✗ excluded
├── node_modules/         ✗ excluded
├── *.log                 ✗ excluded
└── .env                  ✗ excluded
```

This is important for:

* performance
* reducing unnecessary data
* preventing accidental inclusion of files
* keeping sensitive files out of the build context

We don't need to study `.dockerignore` deeply yet, but you should understand **why it exists**.

---

# 17. Beginner Misconception: `.dockerignore` Is the Same as `.gitignore`

No.

They serve different systems.

```text
.gitignore
    ↓
Controls what Git tracks

.dockerignore
    ↓
Controls what Docker excludes from build context
```

For example:

```text
.gitignore
```

might exclude:

```text
node_modules/
```

but that does not automatically mean Docker's build context behavior is identical.

You should think of them independently.

---

# 18. Build Context and Security

This is another important practical concept.

Suppose your project contains:

```text
my-app/
├── Dockerfile
├── app.py
├── .env
├── private-key.pem
└── database.sql
```

If you use:

```bash
docker build -t my-app:1.0 .
```

without considering your context and `.dockerignore`, those files can become part of the build input.

Even if you don't explicitly:

```dockerfile
COPY . .
```

you should still avoid unnecessarily making sensitive material available to the build.

A good mental model is:

> **Only put what the build needs into the build context, and exclude sensitive/unnecessary files.**

---

# 19. `COPY . .` — Why It Can Be Dangerous

You will often see:

```dockerfile
COPY . .
```

This means roughly:

> Copy the relevant contents of the build context into the current destination.

Suppose the context is:

```text
my-app/
├── Dockerfile
├── app.py
├── requirements.txt
├── .git/
└── .env
```

Then:

```dockerfile
COPY . .
```

can potentially bring a lot of that context into the image unless exclusions are applied.

This is why `.dockerignore` becomes important.

We'll study this more deeply later.

For now:

```text
COPY . .
    ↓
Potentially copy a large portion of build context
```

So don't blindly assume:

> "`COPY . .` means copy my application."

It really means:

> **Copy from the build context.**

---

# 20. What Docker Is Actually Doing

Let's make the complete process concrete.

Suppose:

```text
my-app/
├── Dockerfile
├── app.py
└── requirements.txt
```

You run:

```bash
docker build -t my-app:1.0 .
```

Conceptually:

```text
Step 1
Docker receives the build request

        ↓

Step 2
Docker identifies the Dockerfile

        ↓

Step 3
Docker identifies "."
as the build context

        ↓

Step 4
Build input is prepared from
that context

        ↓

Step 5
Docker processes the Dockerfile

        ↓

Step 6
COPY instructions can access
files from the context

        ↓

Step 7
Docker produces the image
```

The important relationship is:

```text
                 docker build
                     │
            ┌────────┴────────┐
            │                 │
            ▼                 ▼
       Dockerfile        Build Context
       instructions       available files
            │                 │
            └────────┬────────┘
                     ▼
                 Build Process
                     │
                     ▼
                 Docker Image
```

---

# 21. Build Context Is Determined by the Command

Consider:

```bash
docker build -t my-app:1.0 .
```

Context:

```text
.
```

Now:

```bash
docker build -t my-app:1.0 ./backend
```

Context:

```text
./backend
```

And:

```bash
docker build -t my-app:1.0 /home/user/project
```

Context:

```text
/home/user/project
```

So the final argument to `docker build` is extremely important.

It tells Docker:

> **This is the build context.**

---

# 22. Example: Different Contexts

Suppose:

```text
project/
├── backend/
│   ├── Dockerfile
│   └── app.py
│
└── frontend/
    ├── package.json
    └── app.js
```

If you run:

```bash
docker build -t backend:1.0 ./backend
```

the context is:

```text
backend/
```

So the build sees:

```text
backend/
├── Dockerfile
└── app.py
```

It doesn't automatically get:

```text
frontend/
```

because:

```text
frontend/
```

is outside the selected context.

This is a very useful concept for monorepos and CI/CD pipelines.

---

# 23. A Common Error

Suppose your context is:

```text
backend/
```

and your Dockerfile says:

```dockerfile
COPY ../frontend/app.js /app/
```

A beginner might think:

> "The file exists, so Docker should copy it."

But Docker's build boundary matters.

The file is outside the build context.

So the problem isn't:

```text
Does the file exist?
```

The more important question is:

```text
Is the file available inside the build context?
```

This distinction is fundamental.

---

# 24. Build Context and CI/CD

Now let's connect this to your CI/CD goal.

A Jenkins pipeline may execute:

```bash
git clone <repository>
cd my-app
docker build -t my-app:${BUILD_NUMBER} .
```

Here:

```text
Git repository
      │
      ▼
Jenkins workspace
      │
      ▼
Current directory
      │
      ▼
Build context
      │
      ▼
Docker build
      │
      ▼
Image
```

So the CI server's workspace effectively becomes the source of the build context.

This is why the structure of your Git repository matters to Docker builds.

---

# 25. Build Context in a Real CI Pipeline

Imagine:

```text
Jenkins workspace
│
├── Dockerfile
├── src/
├── requirements.txt
├── tests/
├── .git/
└── documentation/
```

Jenkins runs:

```bash
docker build -t payment-service:152 .
```

The context is:

```text
Jenkins workspace/
```

Docker then processes the Dockerfile against that context.

A well-designed `.dockerignore` can prevent unnecessary files from becoming build input.

Conceptually:

```text
Jenkins workspace
        │
        │ build context
        ▼
┌─────────────────────┐
│ Docker build        │
│                     │
│ Dockerfile          │
│ source files        │
│ required files      │
└─────────────────────┘
        │
        ▼
payment-service:152
```

This is a very common CI/CD pattern.

---

# 26. An Important Distinction: Context vs Image

Let's compare them directly.

### Build context

```text
Input to the build
```

Example:

```text
app.py
requirements.txt
Dockerfile
```

### Docker image

```text
Output of the build
```

Example:

```text
my-app:1.0
```

So:

```text
Build Context
     │
     │ input
     ▼
Docker Build
     │
     │ output
     ▼
Docker Image
```

Don't confuse:

```text
build context ≠ image
```

---

# 27. An Important Distinction: Context vs Container

Similarly:

```text
Build Context
     │
     ▼
Docker Build
     │
     ▼
Image
     │
     ▼
Container
```

The build context is used **during image creation**.

It isn't a directory mounted into the running container.

This is a very important distinction.

For example:

```text
my-app/app.py
```

being part of the build context does **not** mean the running container automatically has access to the host directory.

The file has to be included in the image or provided through another runtime mechanism.

We'll study container filesystems later.

---

# 28. Beginner Misconception: Build Context Is a Volume

No.

Build context is not:

```text
host directory
     ↕
container directory
```

That's a mount/volume concept.

Build context is:

```text
files available to build
        ↓
Docker build
        ↓
image
```

So:

```text
Build Context
≠
Volume
≠
Bind Mount
```

Those are different concepts.

---

# 29. Beginner Misconception: Everything in Context Goes Into the Image

No.

For example:

```text
Build context
├── app.py
├── test.py
├── README.md
└── Dockerfile
```

Dockerfile:

```dockerfile
COPY app.py /app/
```

Result:

```text
Image
└── app/
    └── app.py
```

The other files don't automatically appear in the image.

The Dockerfile determines what gets used.

So:

```text
Context
= available input

Dockerfile
= instructions controlling the build
```

---

# 30. Beginner Misconception: Bigger Context Means Better Build

Not at all.

Suppose your context is:

```text
5 GB
```

but your application needs only:

```text
100 MB
```

That's unnecessary.

A well-designed build should generally provide only the files required for the build.

This improves:

* build performance
* efficiency
* security
* clarity

This is one reason `.dockerignore` is important.

---

# 31. Hands-On Exercise

Let's create:

```text
context-demo/
├── Dockerfile
├── app.py
├── notes.txt
└── secret.txt
```

`app.py`:

```python
print("Hello from build context")
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
docker build -t context-demo:1.0 .
```

Notice:

```text
Build context:
context-demo/
```

It contains:

```text
Dockerfile
app.py
notes.txt
secret.txt
```

But the Dockerfile only says:

```dockerfile
COPY app.py .
```

So the image receives `app.py`, not automatically every file in the context.

---

# 32. Now Change the Context

Create:

```text
context-demo/
├── Dockerfile
├── app.py
└── files/
    └── message.txt
```

Dockerfile:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY files/message.txt .

CMD ["cat", "message.txt"]
```

Build:

```bash
docker build -t context-demo:2.0 .
```

This works because:

```text
files/message.txt
```

is inside:

```text
.
```

the build context.

---

# 33. Now Understand the Boundary

Imagine:

```text
workspace/
├── shared.txt
│
└── context-demo/
    ├── Dockerfile
    └── app.py
```

If you run from `context-demo`:

```bash
docker build -t context-demo:3.0 .
```

then:

```text
Build context
└── context-demo/
```

not:

```text
workspace/
```

Therefore:

```text
workspace/shared.txt
```

is outside the context.

This experiment is worth doing because it makes the boundary intuitive.

---

# 34. The Build Context Mental Model

At this point, think of:

```text
docker build -t my-app:1.0 .
```

as:

```text
                  docker build
                       │
          ┌────────────┴────────────┐
          │                         │
          ▼                         ▼
     Dockerfile              Build Context
     "What to do"             "What is available"
          │                         │
          └────────────┬────────────┘
                       ▼
                  Build Process
                       │
                       ▼
                  Docker Image
```

This is the mental model you should retain.

---

# 35. How Step 2 and Step 3 Fit Together

We can now connect the previous two topics.

### Step 2 — Dockerfile

We learned:

> **What should Docker do?**

### Step 3 — Build Context

We learn:

> **What files are available for Docker to use while doing it?**

Together:

```text
Dockerfile
"Instructions"
       +
Build Context
"Available build input"
       │
       ▼
Docker Build
       │
       ▼
Docker Image
```

This is much more powerful than thinking of them as two unrelated Docker concepts.

---

# 36. What We Are NOT Learning Yet

To keep the roadmap clean, we're **not going deeply into**:

* image layers
* build cache
* BuildKit internals
* multi-stage builds
* registry
* image tags
* container filesystem
* base images
* advanced `.dockerignore`
* remote build contexts
* Git/URL contexts
* named contexts

Some of these are related to build context, but we'll encounter them at the appropriate level later.

The next major concept is:

> **Docker image layers**

That's where we'll answer:

> **When Docker processes all these Dockerfile instructions, how does the resulting image actually get constructed?**

---

# 37. Beginner vs Advanced

### You should understand now

You should be comfortable with:

* What a build context is
* What `.` means in `docker build ... .`
* The difference between Dockerfile and build context
* Why `COPY` depends on the build context
* Why files outside the context aren't normally available to `COPY`
* Context vs image
* Context vs container
* Why `.dockerignore` exists
* Why large contexts can hurt build efficiency
* Why build context matters in CI/CD
* The fact that not everything in the context automatically goes into the image

### Advanced — leave for later

Don't worry yet about:

* BuildKit internals
* named contexts
* remote contexts
* Git contexts
* context transfer implementation
* advanced `.dockerignore` pattern behavior
* multi-stage build context optimization
* BuildKit secrets and mounts

Those will make more sense after the core image-building model is established.

---

# 38. The One-Sentence Takeaway

If you remember only one thing from Step 3:

> **The Docker build context is the set of files and directories made available to a Docker build, and the final argument to `docker build` specifies which location is used as that context.**

So when you run:

```bash
docker build -t my-app:1.0 .
```

the mental model is:

```text
. 
│
└── "Use this directory as the build context"
                 │
                 ▼
           Docker Build
                 │
       ┌─────────┴─────────┐
       │                   │
 Dockerfile          Context files
 "instructions"       "available input"
       │                   │
       └─────────┬─────────┘
                 ▼
             Image
```

---

# Phase 1 Progress

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

And now the sequence becomes much more meaningful:

```text
Dockerfile
    │
    │ tells Docker what to do
    │
    ▼
Build Context
    │
    │ provides files Docker can use
    │
    ▼
Docker Build
    │
    │ processes instructions
    ▼
Docker Image Layers
    │
    │ eventually compose
    ▼
Docker Image
```

**Next: Step 4 — Docker Image Layers.**
