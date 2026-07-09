# 06 - Utility Containers

## What is a Utility Container?

Up until this point, every container we built was an **Application Container** — a container that packages an entire application (a Node.js app, a React app, a database, etc.) along with everything it needs to run. When the container starts, it automatically executes the app.

A **Utility Container** (not an official Docker term, just a useful way to describe the pattern) is different: instead of packaging a full application, it only packages an **environment/toolset** (e.g. just Node.js, just PHP). It doesn't run anything automatically — *you* decide what command to execute inside it.

This turns Docker into more than a tool for running apps — it becomes a tool for running **commands and tasks** inside isolated environments, without installing those tools on your host machine.

### Why this matters

Creating a new project often requires tools installed locally:
- A new Node.js project needs `npm init`, which needs Node.js/npm installed.
- A new Laravel (PHP) project needs PHP, Composer, and several other tools installed.

But the whole point of Docker is to avoid installing tools globally on the host. Utility Containers solve this: instead of installing Node.js or PHP on your machine, you keep them inside a container and run commands through it.

---

## `docker exec` — running extra commands in a running container

`docker exec` lets you run an additional command inside a container that is **already running**, without stopping it.

```bash
# run a Node container in interactive + detached mode
docker run -it -d node

# check it's running
docker ps

# execute an extra command inside it (needs -it to answer prompts)
docker exec -it <container-name> npm init
```

Useful for things like inspecting logs, running maintenance/debug commands, etc., without touching the main process.

---

## Overriding the default command (`CMD`)

Every image has a default command. For the `node` image, that default is starting the Node REPL.

You can override it by adding a command after the image name:

```bash
docker run -it node npm init
```

Here, instead of dropping into the Node REPL, the container runs `npm init` and then stops once that command finishes (since there's no longer a long-running process keeping it alive).

---

## Building a Node "utility" image

**Dockerfile**
```dockerfile
FROM node:14-alpine
WORKDIR /app
```

- `node:14-alpine` — Node 14 built on Alpine Linux, a very small/lightweight base image (~35MB), ideal for a utility image.
- `WORKDIR /app` — sets the folder any command will run from inside the container.
- No `CMD` is set on purpose — the whole point is to let the user run *any* command they want (`npm init`, `npm install`, `npm test`, etc.), not lock the image to one task.

Build it:
```bash
docker build -t node-util .
```

Run it with a **bind mount** so files created inside the container (like `package.json`) actually appear on your host machine, not just inside the container:

```bash
docker run -it -v $(pwd):/app node-util npm init
```

Now you can `npm init` a project without Node.js installed on your host at all — everything runs inside the container, and results are written back to your project folder via the bind mount.

---

## `ENTRYPOINT` vs `CMD`

- **`CMD`**: if you pass a command after the image name in `docker run`, it **overrides** the `CMD`.
- **`ENTRYPOINT`**: if you pass a command after the image name, it gets **appended as arguments** to the entrypoint instead of replacing it.

```dockerfile
FROM node:14-alpine
WORKDIR /app
ENTRYPOINT ["npm"]
```

Build it as `mynpm`. Now every argument you pass becomes an `npm` subcommand:

```bash
docker run -it -v $(pwd):/app mynpm init      # runs: npm init
docker run -it -v $(pwd):/app mynpm install   # runs: npm install
docker run mynpm install express --save       # runs: npm install express --save
docker run mynpm test                         # runs: npm test
```

This locks the image specifically to `npm` commands — which is also a safety benefit: it prevents the container (and by extension, the bind-mounted host folder) from running arbitrary/dangerous commands.

---

## Using Docker Compose for Utility Containers

Typing the full `docker run -it -v ... image command` every time gets tedious. Docker Compose can store all of that configuration — even for a single-service, non-persistent utility container.

**docker-compose.yaml**
```yaml
version: "3.8"
services:
  npm:
    build: ./
    stdin_open: true   # same as -it (interactive input)
    tty: true
    volumes:
      - ./:/app
```

### `docker compose up` vs `docker compose run`

- `docker compose up` is meant for **application containers** meant to stay running (web servers, databases, APIs). It doesn't accept a one-off command argument.
- `docker compose run <service> <command>` is meant for **one-off tasks**: it starts the service, runs the given command, and stops the container once done — exactly what a utility container needs.

```bash
docker compose run npm init
docker compose run npm install express --save
```

### Cleaning up

`docker compose run` does **not** automatically remove the container after it exits (unlike `docker run --rm`). Leftover stopped containers pile up over time. Fix it with `--rm`:

```bash
docker compose run --rm npm init
```

Or clean up manually anytime with:
```bash
docker container prune
```

---

## Key takeaways

- **Application Containers** run a full app and stay alive; **Utility Containers** just provide an environment/toolset to execute one-off commands.
- `docker exec` runs extra commands inside an already-running container.
- Passing a command after the image name overrides `CMD`, but is appended as an argument to `ENTRYPOINT`.
- `ENTRYPOINT` is great for building a "wrapper" image focused on one tool (e.g. `npm`), and adds a layer of safety by restricting what can be run.
- Bind mounts (`-v`) are what make Utility Containers practical — they let files created inside the container (like `package.json`, `node_modules`) show up on the host.
- With Docker Compose, use `docker compose run --rm <service> <command>` for utility/one-off tasks instead of `docker compose up`.
- This pattern is especially valuable for frameworks with heavier setup requirements (e.g. **Laravel/PHP**), where you'd otherwise need to install many tools locally just to scaffold a project.