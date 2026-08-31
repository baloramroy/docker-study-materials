# Phase 2 — Build Your First Docker Image

## Part 5 — Assemble, Build & Run

### Where We Are

We've finished the instruction-level teaching:

```text
FROM
WORKDIR
COPY
RUN
ENV
EXPOSE
ENTRYPOINT
CMD
```

and established the key distinction:

```text
RUN  → BUILD TIME
CMD  → RUNTIME
```

Now we actually build the image. This part is different from the last few — instead of learning instructions one at a time, we assemble what we've learned and watch Docker turn our application into a real image.

---

# 1. Our Project

```text
my-app/
├── app.py
└── Dockerfile
```

```python
print("Hello from Docker!")
```

```dockerfile
FROM python:3.12

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

```text
app.py + Dockerfile → docker build → Image (my-app:1.0) → docker run → Container → Hello from Docker!
```

Let's do it for real.

---

# 2. Build the Image

```bash
cd my-app
ls
```

You should see `Dockerfile` and `app.py`. Now:

```bash
docker build -t my-app:1.0 .
```

This is the first real build of the phase. Breaking the command down:

```text
docker        → the CLI
build         → build an image
-t my-app:1.0 → name and tag the resulting image
.             → the build context (Phase 1 callback: current directory)
```

> Build an image using the current directory as the build context, and name the result `my-app:1.0`.

---

# 3. What Docker Does During the Build

```text
Read Dockerfile
     ↓
FROM python:3.12   → start from base
     ↓
WORKDIR /app       → set working directory
     ↓
COPY app.py .      → add application
     ↓
CMD [...]          → record default command (does NOT execute it)
     ↓
my-app:1.0
```

Worth repeating: `CMD` does not run `python app.py` during the build. It only *records* the default runtime command. The application runs later, once a container exists.

The exact build output varies by Docker version and cache state — don't worry if yours doesn't match an example exactly. What matters is ending up with `my-app:1.0`.

---

# 4. Verify and Inspect

```bash
docker images
```

```text
REPOSITORY   TAG   IMAGE ID    SIZE
my-app       1.0   abc123...   ...
```

`my-app:1.0` — image name and tag (Phase 1 callback: same `name:tag` format as any other image reference).

Now look inside it:

```bash
docker image inspect my-app:1.0
```

This returns a large JSON block — don't try to absorb all of it. Two fields are worth finding on purpose, because they trace directly back to our Dockerfile:

```text
WorkingDir → should read "/app"           (from WORKDIR /app)
Cmd        → should read ["python","app.py"]  (from CMD [...])
```

Seeing your own Dockerfile reflected in the image's actual configuration is the point of this step — it confirms the instructions did what you expect, and reinforces that `Cmd` here is *stored*, not *already run*.

You will see information including things related to:

```text
Image ID
Created time
Architecture
OS
Config
Environment
Working directory
Command
Entrypoint
Root filesystem
Layers
```

This is why `docker image inspect` is useful:

> **It lets us examine how Docker understands the image.**

---

# 5. Run It

```bash
docker run my-app:1.0
```

```text
Image (my-app:1.0) → docker run → Container → CMD fires → python app.py → "Hello from Docker!"
```

Docker didn't rebuild anything here — the image already existed. `docker run` only creates a new container from it and starts its default command.

You'll notice the container prints the message and then your terminal returns immediately — the container has already stopped. That's expected, and it's the subject of the next part (Part 6) in detail: for now, just know it's because the main process (`python app.py`) finished and exited normally, not because anything went wrong.

---

# 6. What Each Command Does

| Command | Purpose |
|---|---|
| `docker build -t my-app:1.0 .` | Creates the image from the Dockerfile + build context |
| `docker images` | Lists images available locally |
| `docker image inspect my-app:1.0` | Shows the image's detailed metadata/configuration |
| `docker run my-app:1.0` | Creates and starts a container from the image |

---

# 7. Why This Matters

Notice what we *didn't* do on the host: no manually installing Python, creating `/app`, copying the file by hand, or remembering a startup command. All of that was described once, in the Dockerfile, and Docker turned it into a reusable image:

> **The Dockerfile turns a repeatable set of instructions into a reusable artifact.**

If you build again right now without changing anything —

```bash
docker build -t my-app:1.0 .
```

— Docker may reuse previously-built layers instead of redoing identical work (Phase 1 callback: this is the layer/cache behavior from Part 4 of Phase 1). You don't need to study cache optimization yet — just notice that Docker isn't necessarily repeating unchanged steps from scratch.

---

# Part 5 Checkpoint

You should be able to run this whole workflow yourself and explain each stage:

```text
docker build          → produces the IMAGE
docker images         → lists it
docker image inspect  → lets you examine it
docker run            → produces a CONTAINER from it
```

```bash
docker build -t my-app:1.0 .
docker images
docker image inspect my-app:1.0
docker run my-app:1.0
```

Expected output: `Hello from Docker!`

**Part 5 is complete.** We've moved from *learning* Dockerfile instructions to actually *building and running* an image.

In **Part 6 — What Actually Happened**, we'll properly examine why the container stopped, look at image vs. container more concretely, and finish with a hands-on exercise that serves as the Phase 2 checkpoint.