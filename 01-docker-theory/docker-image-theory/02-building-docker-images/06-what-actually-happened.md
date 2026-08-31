# Phase 2 — Build Your First Docker Image

## Part 6 — What Actually Happened

### Where We Are

We've now built and run our first Docker image.

```dockerfile
FROM python:3.12

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

```bash
docker build -t my-app:1.0 .
docker run my-app:1.0
```

output:

```text
Hello from Docker!
```

Now we close the loop. The goal isn't new syntax — it's making sure you understand what actually happened, end to end, and why the container behaved the way it did.

---

# 1. Reconstructing the Whole Journey

```text
app.py                    (source)
   +
Dockerfile                (instructions)
   │
   │ docker build
   ▼
Docker Image
my-app:1.0
   │
   │ docker run
   ▼
Container created
   │
   ▼
Main process starts: python app.py
   │
   ▼
"Hello from Docker!" printed
   │
   ▼
Process exits
   │
   ▼
Container stops — Exited (0)
```

Each Dockerfile instruction played a specific role in producing the image:

```dockerfile
FROM python:3.12          → starting environment
WORKDIR /app               → working location
COPY app.py .               → application added
CMD ["python", "app.py"]   → default command recorded (not executed yet)
```

That `CMD` line only became active once `docker run` created a container from the image — everything before it happened at build time; `CMD` is what fired at runtime.

---

# 2. Why Did the Container Stop?

Our application:

```python
print("Hello from Docker!")
```

Python starts, prints the line, and finishes — there's no more work to do, so the process exits, and the container stops with it.

```text
Container starts → python app.py → prints message → process exits → container stops
```

**This is not an error.** The container stopped because its main process completed successfully.

A common beginner expectation is "I ran the container, so it should keep running" — but a container's lifetime is tied to its main process. Compare our script to a web server:

```text
Our script:     start → print → exit           (container lasts moments)
A web server:   start → listen → keep running   (container stays up)
```

Same mechanism, different outcome, because the *process* behaves differently. This distinction matters a lot once you start containerizing real services.

---

# 3. Image vs Container — One More Time, Concretely

Be precise with terminology here:

- The **container** stopped.
- The **image** did not.

```bash
docker images
```

still shows:

```text
my-app   1.0
```

The image is a reusable template; a container is one runtime instance of it. You could run `docker run my-app:1.0` again right now and get a brand new container, independent of the one that already exited.

---

# 4. Finding a Stopped Container

```bash
docker ps
```

only shows **running** containers — so our exited one won't appear there.

```bash
docker ps -a
```

shows **all** containers, running or stopped:

```text
CONTAINER ID   IMAGE        STATUS
abc123...      my-app:1.0   Exited (0) ...
```

`Exited (0)` means the main process exited successfully — `0` is a normal exit code, not a failure. You'll reach for `docker ps -a` constantly once you start troubleshooting containers that "disappeared."

---

# 5. All Eight Instructions — Final Reference

| Instruction  | Purpose                                        | When it acts     |
| ------------ | ----------------------------------------------- | ---------------- |
| `FROM`       | Select the starting/base image                  | Build            |
| `WORKDIR`    | Set the working directory                       | Build            |
| `COPY`       | Copy files from build context into the image    | Build            |
| `RUN`        | Execute commands during image build             | Build            |
| `ENV`        | Define environment variables                    | Build → runtime  |
| `EXPOSE`     | Declare an intended container port              | Metadata only    |
| `ENTRYPOINT` | Define the main executable                      | Runtime          |
| `CMD`        | Define the default runtime command/arguments    | Runtime          |

Our image only used `FROM`, `WORKDIR`, `COPY`, and `CMD` — and that's fine. A good Dockerfile uses the instructions an application actually needs, not every instruction that exists.

If you remember one thing from this whole phase, make it this:

```text
Build time:    FROM, WORKDIR, COPY, RUN   → construct the image
Runtime:       ENTRYPOINT, CMD             → define the container's main process
```

---

# 6. Read the Dockerfile Like a Sentence

```dockerfile
FROM python:3.12
WORKDIR /app
COPY app.py .
CMD ["python", "app.py"]
```

> Start with Python 3.12. Work inside `/app`. Put `app.py` there. When a container starts, run `python app.py`.

That's the bar we're aiming for — not "I memorized four commands," but "I understand what Docker is being told to do."

---

# 7. Phase 2 Hands-On Exercise

Build a *different* image, without copying the lesson's exact example.

```text
docker-practice/
├── app.py
└── Dockerfile
```

**`app.py`** — print three lines of your own choosing (not the lesson's examples):

```text
Docker is working!
I built my first image.
I understand containers.
```

**Dockerfile requirements** — use `FROM`, `WORKDIR`, `COPY`, `CMD` to:

1. Start from a Python image.
2. Set a working directory.
3. Copy your application in.
4. Run it by default when the container starts.

Then:

```bash
docker build -t docker-practice:1.0 .
docker images
docker image inspect docker-practice:1.0
docker run docker-practice:1.0
```

You should see your three lines printed.

---

# 8. Your Checkpoint

Don't just report "it worked." Be able to explain **why**:

1. **Why does `FROM` come first?** The image needs a starting/base environment before anything else can be added.
2. **Why does `COPY` put the application inside the image?** The container's filesystem needs to actually contain the app to run it.
3. **Why does `CMD` contain your run command?** It's the default process Docker starts when a container is created from the image.
4. **Why does the container stop after printing?** The main process finishes — no more work, no more process, no more container.
5. **What's the difference between image and container?** Image = reusable template. Container = one running instance of it.

---

# Phase 2 — Complete

If you can complete the exercise and answer the checkpoint, you've hit Phase 2's core objective:

> You can take a simple application, describe how its image should be built with a Dockerfile, build that image, inspect it, and run a container from it.

```text
Application → Dockerfile → docker build → Image → docker run → Container → Process → Application runs
```

The next phase should build on this rather than repeat it — moving toward more realistic applications, dependencies, container networking, persistent data, and the features needed to run something beyond a single `print()` process.