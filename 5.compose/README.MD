# Project 5 – Docker Compose

Part of my **Learning Docker** journey. This project takes the multi-container app from the previous project (Node.js backend + React frontend + MongoDB, each run manually with `docker build` / `docker run`) and rebuilds it using **Docker Compose** to manage everything with a single configuration file.

## 📌 About

In the previous project, the app was made of three separate containers, each started with long, manual `docker run` commands (environment variables, volumes, networks, ports — all typed out by hand every time). This project replaces all of that with one file: `docker-compose.yml`.

**The app consists of:**
- **Backend** – Node.js / Express REST API
- **Frontend** – React Single Page Application
- **Database** – MongoDB

## 🧠 What is Docker Compose?

Docker Compose is a tool that lets you define a multi-container application in a single YAML file (`docker-compose.yml`) instead of writing/copy-pasting long `docker build` and `docker run` commands every time.

- It does **not** replace the `Dockerfile` — it still uses it to build custom images.
- It does **not** create a new kind of image or container — it just automates managing existing ones.
- It's designed for containers running on a **single host** (not for orchestrating containers across multiple machines/servers — that's a separate topic, e.g. deployment/orchestration tools).
- One command starts everything (`docker-compose up`), and one command stops/removes everything (`docker-compose down`).

## 🗂️ Project Structure

```
.
├── docker-compose.yaml
├── backend/
│   ├── Dockerfile
│   └── ... (Node/Express app)
├── frontend/
│   ├── Dockerfile
│   └── ... (React app)
└── env/
    ├── mongo.env
    └── backend.env
```

## ⚙️ docker-compose.yaml Breakdown

### `version`
Declares which Docker Compose specification version the file follows (e.g. `"3.8"`), so Compose knows what syntax/features are supported.

### `services`
The core of the file. Each service represents one container:
- `mongodb`
- `backend`
- `frontend`

### MongoDB service
- `image: "mongo"` – uses the official MongoDB image from Docker Hub (no custom Dockerfile needed).
- `volumes: - data:/data/db` – a **named volume** to persist database data.
- `environment` – sets `MONGO_INITDB_ROOT_USERNAME` and `MONGO_INITDB_ROOT_PASSWORD` (can be written directly, as a list with `=`, or pulled from an external `.env` file via `env_file`).
- No manual network setup needed — Compose automatically creates a **default network** and connects all services to it.

### Backend service
- `build: ./backend` – instead of a ready-made image, Compose builds the image itself from the Dockerfile in the `backend/` folder. (Longer form available too: `context` + `dockerfile` + `args` if needed.)
- `ports: - "80:80"` – maps host port to container port.
- `volumes`:
  - Named volume for logs
  - **Bind mount** `./backend:/app` (relative path instead of the long absolute path required by plain `docker run`) so code changes reflect live inside the container
  - Anonymous volume (e.g. to protect `node_modules`)
- `env_file: - ./env/backend.env` – keeps sensitive variables (DB username/password) out of the compose file and out of Git (added to `.gitignore`).
- `depends_on: - mongodb` – ensures MongoDB starts before the backend, since the API needs to connect to it.

### Frontend service
- `build: ./frontend` – builds from the React app's Dockerfile.
- `ports: - "3000:3000"`
- `volumes: - ./frontend/src:/app/src` – bind mount so local code edits show up immediately without rebuilding.
- `stdin_open: true` and `tty: true` – replicate the `-it` (interactive + TTY) flags needed for the React dev server to run correctly.
- Optionally `depends_on: - backend`.

### Top-level `volumes`
Named volumes (like `data` for MongoDB) must be declared once more at the bottom of the file, outside `services`, so Docker knows to create/manage them.

## 🚀 Getting Started

### Prerequisites
- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)

### Run the project
```bash
# Start everything (builds images if missing, pulls MongoDB, starts all containers)
docker-compose up

# Start in the background (detached mode)
docker-compose up -d

# Force a rebuild of images (e.g. after changing a Dockerfile)
docker-compose up --build

# Only build the custom images defined with "build:" (without starting containers)
docker-compose build
```

### Stop the project
```bash
# Stop and remove containers + the default network (volumes are kept!)
docker-compose down

# Stop and remove containers + network + volumes (data will be lost)
docker-compose down -v
```

## 🏷️ Naming notes

- The names under `services:` (`mongodb`, `backend`, `frontend`) are **service names**, not the actual container names.
- Actual container names follow the pattern: `<project_folder_name>_<service_name>_<number>` (e.g. `docker-complete_backend_1`).
- **Service names are what matter for networking** — the app code (e.g. `app.js`) connects to MongoDB using the service name `mongodb`, and Compose's internal DNS resolves it correctly regardless of the real container name.
- A custom, fixed container name can be set with `container_name: mongodb` if needed.

## 🧪 Useful Commands

```bash
docker-compose up          # start all services (foreground, shows logs)
docker-compose up -d       # start all services (detached)
docker-compose down        # stop & remove containers + network (keeps volumes)
docker-compose down -v     # stop & remove containers + network + volumes
docker-compose build       # build custom images only
docker-compose up --build  # force rebuild before starting
docker ps                  # list running containers
docker volume ls           # list all volumes (named, anonymous, Compose-created)
docker logs <container>    # view logs for a specific container
```

## 📚 What I Learned

- Docker Compose replaces long, repetitive `docker build` / `docker run` commands with one declarative `docker-compose.yaml` file.
- `services` is the core section — each service maps to one container, with its own `image`/`build`, `ports`, `volumes`, and `environment`.
- Compose automatically creates a shared **default network**, so containers can talk to each other using their **service names** — no manual network setup required.
- Bind mounts use simple **relative paths** in Compose instead of long absolute paths.
- `depends_on` controls startup order between dependent services (e.g. backend waits for the database).
- `docker-compose down` removes containers and networks but **preserves named volumes** by default — data isn't lost between restarts, unless `-v` is explicitly used.
- Compose doesn't replace `Dockerfile`, `docker build`, or `docker run` — it works with them and runs them automatically under the hood.
- Even single-container projects can benefit from Compose, since keeping configuration in one file is easier to maintain than long terminal commands.



## 📄 License

This project is for educational purposes.