# Phase 5 — Multi-Stage Builds

## Part 1: The Problem Multi-Stage Builds Solve

Phase 4 taught us how to optimize a **single-stage image**: choose an appropriate base image, avoid unnecessary packages, clean temporary data, and run as a non-root user.

Now we address a bigger problem.

> **What if the tools required to build an application are not required to run the application?**

That is exactly what multi-stage builds solve.

---

## 1. The Build Environment vs Runtime Environment

Consider a Go application.

To build it, you might need:

```text
Go compiler
Go modules
source code
build tools
```

But after compilation, the resulting application may be just:

```text
myapp
```

The production container doesn't need the Go compiler.

Yet with a normal single-stage Dockerfile, you might end up with everything:

```text
Go compiler
Go tooling
source code
dependencies
myapp
```

That creates a larger runtime image than necessary.

The desired architecture is:

```text
                 Docker build
                      |
          +-----------+-----------+
          |                       |
          v                       v
     Build stage              Runtime stage
          |                       |
     Go compiler              Minimal image
     Source code              myapp binary
     Build tools                   |
          |                        |
          +------ COPY ------------+
                   |
                   v
              Final image
```

The key idea is:

> **Use one environment to build the application and another environment to run it.**

---

# 2. A Single-Stage Example

Let's use a simple Go application.

`main.go`:

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello from Docker!")
}
```

A single-stage Dockerfile could look like:

```dockerfile
FROM golang:1.25

WORKDIR /app

COPY . .

RUN go build -o myapp .

CMD ["./myapp"]
```

This works.

But look at the final image.

The application needs:

```text
myapp
```

The image also contains the Go environment because the final image is based on:

```text
golang:1.25
```

The compiler was necessary **during the build**, but not necessarily **during runtime**.

That's the inefficiency.

---

# 3. The Multi-Stage Solution

We can split the Dockerfile into two stages:

```dockerfile
FROM golang:1.25 AS builder

WORKDIR /app

COPY . .

RUN go build -o myapp .
```

Then start a new stage:

```dockerfile
FROM debian:bookworm-slim

WORKDIR /app

COPY --from=builder /app/myapp .

CMD ["./myapp"]
```

The complete Dockerfile is:

```dockerfile
FROM golang:1.25 AS builder

WORKDIR /app

COPY . .

RUN go build -o myapp .


FROM debian:bookworm-slim

WORKDIR /app

COPY --from=builder /app/myapp .

CMD ["./myapp"]
```

There are now two stages:

```text
Stage 1
golang:1.25
     |
     ├── source code
     ├── compiler
     ├── build tools
     └── myapp
             |
             | COPY --from=builder
             v
Stage 2
debian:bookworm-slim
     |
     └── myapp
```

Only the **second stage** becomes the final image.

The Go compiler does not come along.

---

# 4. Understanding `AS`

This line:

```dockerfile
FROM golang:1.25 AS builder
```

does two things:

1. Starts a new build stage using `golang:1.25`.
2. Gives that stage the name `builder`.

The name lets us refer to that stage later:

```dockerfile
COPY --from=builder /app/myapp .
```

Think of:

```text
AS builder
```

as giving the stage a label.

For example:

```dockerfile
FROM node:22 AS builder
```

creates:

```text
builder
```

Then:

```dockerfile
COPY --from=builder ...
```

means:

> Copy something from the filesystem produced by the `builder` stage.

---

# 5. Understanding `COPY --from`

This is the most important new Dockerfile instruction in this phase.

Normal:

```dockerfile
COPY app.py .
```

means:

> Copy from the build context.

But:

```dockerfile
COPY --from=builder /app/myapp .
```

means:

> Copy `/app/myapp` from the `builder` stage into the current stage.

So there are now two different sources you need to distinguish:

```text
COPY
  |
  └── Build context


COPY --from=builder
  |
  └── Previous build stage
```

This distinction is fundamental to understanding multi-stage builds.

---

# 6. What Becomes the Final Image?

This is another important point.

Suppose the Dockerfile has:

```dockerfile
FROM golang:1.25 AS builder

# build...


FROM debian:bookworm-slim

# runtime...
```

The final image is based on:

```text
debian:bookworm-slim
```

**not**:

```text
golang:1.25
```

The builder stage is used to produce artifacts.

The last stage defines the runtime image.

Conceptually:

```text
Stage 1                  Stage 2
────────                 ────────
golang                   debian
   │                         │
   │ build                   │
   ▼                         │
myapp ──────────────────────►│
                             │
                             ▼
                         FINAL IMAGE
```

---

# 7. Why This Is Better

The application only needs the runtime environment.

Instead of:

```text
Final image
├── Go compiler
├── Go tooling
├── Source code
├── Build dependencies
└── Application
```

we can have:

```text
Final image
├── Runtime libraries
└── Application
```

This can provide several benefits:

* Smaller runtime images.
* Faster image pulls.
* Less unnecessary software.
* Reduced attack surface.
* Cleaner production environments.
* Separation between build and runtime dependencies.

For CI/CD, this is especially useful because the image produced by the pipeline is the artifact that gets pushed to the registry and deployed.

---

# 8. Multi-Stage Builds Are Not Just About Image Size

It's tempting to think:

> Multi-stage build = smaller image.

That's true, but incomplete.

The more important architectural idea is:

> **Build dependencies and runtime dependencies have different purposes.**

For example:

```text
Build dependencies:
- Compiler
- SDK
- Build tools
- Source code
- Test tooling

Runtime:
- Application
- Runtime libraries
- Required configuration
```

Multi-stage builds let you explicitly separate them.

This is why they're so common in production Dockerfiles.

---

# 9. Practical Exercise

Let's build the Go example ourselves.

### Step 1 — Create the project

```bash
mkdir multi-stage-lab
cd multi-stage-lab
```

Create:

```text
multi-stage-lab/
├── main.go
└── Dockerfile
```

### Step 2 — Create `main.go`

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello from a multi-stage Docker build!")
}
```

### Step 3 — Create the first Dockerfile

Start with the **single-stage** version:

```dockerfile
FROM golang:1.25

WORKDIR /app

COPY . .

RUN go build -o myapp .

CMD ["./myapp"]
```

Build it:

```bash
docker build --no-cache -t multi-stage-lab:single .
```

Check its size:

```bash
docker image ls multi-stage-lab
```

Run it:

```bash
docker run --rm multi-stage-lab:single
```

Expected:

```text
Hello from a multi-stage Docker build!
```

---

## 10. Convert It to a Multi-Stage Build

Now replace the Dockerfile with:

```dockerfile
FROM golang:1.25 AS builder

WORKDIR /app

COPY . .

RUN go build -o myapp .


FROM debian:bookworm-slim

WORKDIR /app

COPY --from=builder /app/myapp .

CMD ["./myapp"]
```

Build:

```bash
docker build --no-cache -t multi-stage-lab:multi .
```

Compare:

```bash
docker image ls multi-stage-lab
```

You should see two tags:

```text
multi-stage-lab:single
multi-stage-lab:multi
```

Compare their sizes.

Then run:

```bash
docker run --rm multi-stage-lab:multi
```

Expected:

```text
Hello from a multi-stage Docker build!
```

---

# 11. Verify That the Compiler Is Gone

This is an important part of the exercise.

Run:

```bash
docker run --rm multi-stage-lab:multi go version
```

You should get an error because the final image does not contain the Go compiler/runtime tooling.

That's exactly what we want.

The builder had:

```text
Go
Compiler
Source code
myapp
```

The final image has:

```text
myapp
```

plus whatever the runtime base requires.

You can also inspect the final image:

```bash
docker run --rm multi-stage-lab:multi ls -la /app
```

You should see the application binary:

```text
myapp
```

---

# 12. The Mental Model You Should Keep

Don't memorize the syntax first.

Understand this:

```text
              BUILD STAGE
          ┌─────────────────┐
          │ Compiler        │
          │ Source code     │
          │ Dependencies    │
          │ Build tools     │
          │                 │
          │      BUILD      │
          │        ↓        │
          │     artifact    │
          └────────┬────────┘
                   │
                   │ COPY --from
                   ▼
          ┌─────────────────┐
          │ RUNTIME STAGE   │
          │                 │
          │ Runtime         │
          │ Application     │
          │                 │
          └────────┬────────┘
                   │
                   ▼
              FINAL IMAGE
```

That is the core concept of multi-stage builds.

---

# Checkpoint

Before moving to the next part, make sure you can answer:

1. Why would a Go compiler be unnecessary in the final runtime image?
2. What does `AS builder` do?
3. What does `COPY --from=builder` mean?
4. Which stage becomes the final image?
5. Why does a multi-stage build separate build dependencies from runtime dependencies?
6. Why is the final image potentially smaller and easier to secure?
7. In our example, why does `docker run ... go version` fail on the final image?

Once this mental model is clear, **Part 2 will move beyond the basic example and show how multi-stage builds work with real application dependencies and more practical production patterns.**
