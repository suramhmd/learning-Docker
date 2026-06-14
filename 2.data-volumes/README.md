# Docker Data Management — Volumes, Bind Mounts, ENV & ARG

A hands-on Node.js project built while learning Docker's second core topic: **data persistence and configuration**. The app is a simple feedback collector that accepts user input and saves it to `.txt` files — making it a perfect playground for understanding how Docker manages data.

---

## What This Project Covers

### 1. Volumes (Named & Anonymous)
Docker containers are stateless by default — any data written inside them disappears when the container stops. This project demonstrates how **volumes** solve that problem.

- **Anonymous volumes** were explored for directories like `/app/node_modules` to prevent them from being overwritten by bind mounts.
- **Named volumes** (e.g., `-v feedback:/app/feedback`) persist the user-submitted feedback files across container restarts and removals.

```bash
docker run -v feedback:/app/feedback ...
```

The `/app/feedback` directory inside the container is backed by a Docker-managed volume, so submitted feedback survives even if the container is deleted.

---

### 2. Bind Mounts
Bind mounts connect a **host machine directory** directly into the container. This is the key tool for development workflows — any code change on the host is instantly reflected inside the running container, without rebuilding the image.

```bash
docker run -v $(pwd):/app ...
```

Combined with `nodemon` (configured as the start script), the server reloads automatically on every file save — no rebuild needed during development.

> **Note:** An anonymous volume for `/app/node_modules` is used alongside the bind mount to prevent the host directory from overwriting the container's installed dependencies.

```bash
docker run -v $(pwd):/app -v /app/node_modules ...
```

---

### 3. Environment Variables (`ENV`)
The server port is not hardcoded. Instead, it reads from an environment variable:

```javascript
app.listen(process.env.PORT);
```

In the `Dockerfile`, a default value is set using `ENV`:

```dockerfile
ARG DEFAULT_PORT=80
ENV PORT $DEFAULT_PORT
EXPOSE $PORT
```

This can be overridden at runtime in two ways:

**Inline flag:**
```bash
docker run --env PORT=8000 ...
```

**Using a `.env` file:**
```bash
docker run --env-file .env ...
```

The included `.env` file sets `PORT=8000`.

---

### 4. Build Arguments (`ARG`)
`ARG` allows passing values into the image **at build time** — unlike `ENV`, these are not available at runtime inside the container.

```dockerfile
ARG DEFAULT_PORT=80
ENV PORT $DEFAULT_PORT
```

You can override the default port when building the image:

```bash
docker build --build-arg DEFAULT_PORT=8080 .
```

This makes the image flexible without hardcoding environment-specific values.

---

### 5. `.dockerignore`
The `.dockerignore` file prevents unnecessary files from being copied into the image during `COPY . .`:

```
node_modules
```

This keeps the image lean and avoids conflicts between the host's `node_modules` and the ones installed inside the container via `RUN npm install`.

---

## Project Structure

```
.
├── Dockerfile
├── .dockerignore
├── .env
├── package.json
├── server.js
├── pages/
│   ├── feedback.html
│   └── exists.html
├── public/
├── feedback/       ← persisted via named volume
└── temp/           ← staging area before saving feedback
```

---

## How the App Works

1. The user submits a title and text via a form.
2. The server writes the content to a temp file (`/app/temp/<title>.txt`).
3. It then checks if a file with the same title already exists in `/app/feedback/`.
   - If it **does** → redirect to the "already exists" page.
   - If it **doesn't** → move the file from `temp/` to `feedback/` and redirect home.

---

## Running the App

### Development (with live reload via bind mount)

```bash
docker build -t feedback-app .

docker run \
  --rm \
  -p 3000:80 \
  --name feedback-dev \
  -v feedback:/app/feedback \
  -v $(pwd):/app \
  -v /app/node_modules \
  feedback-app
```

### Production (image only, no bind mount)

```bash
docker run \
  --rm \
  -p 3000:80 \
  --name feedback \
  -v feedback:/app/feedback \
  feedback-app
```

### Custom port via `.env`

```bash
docker run \
  --rm \
  -p 3000:8000 \
  --env-file .env \
  -v feedback:/app/feedback \
  feedback-app
```

---

## Key Takeaways

| Concept | Purpose |
|---|---|
| Named Volume | Persist data across container lifecycles |
| Anonymous Volume | Protect specific container paths from bind mount override |
| Bind Mount | Sync host source code into container for live development |
| `ENV` | Set runtime environment variables with a default in the image |
| `ARG` | Pass build-time variables to customize the image at build |
| `.dockerignore` | Exclude files from the image build context |
