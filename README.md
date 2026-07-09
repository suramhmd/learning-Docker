# 🐳 Learning Docker — From Zero to Confident
A hands-on collection of projects built throughout my Docker learning journey.  
Each folder represents a new concept or milestone — starting from the basics and growing into more complex, real-world use cases.
---
## 📖 About This Course
This repo follows a structured Docker course that covers everything from the ground up — no prior Docker experience required.
Throughout the course I'm learning:
- What Docker is and the core concepts behind it
- What **Containers** are and how they work
- The most important Docker **commands** and what you can do with them
- How to use Docker locally for both simple and complex projects
- How to **deploy** applications using Docker
- How to solve real-world problems with Docker
The course is heavy on **practical examples and demos**, not just theory.  
By the end, the goal is to feel confident using Docker in any new project.
---
## 📁 Projects
| # | Project | Concepts Covered |
|---|---|---|
| 01 | [First Container — Node.js App](./1.first-container/) | Dockerfile, `docker build`, `docker run`, images vs containers, port mapping |
| 02 | [Data & Volumes — Feedback App](./2.data-volumes/) | Named volumes, anonymous volumes, bind mounts, `ENV`, `ARG`, `.dockerignore` |
| 03 | [Networks — SW Favorites API](./3.networks-starting/) | Container networking, `host.docker.internal`, Docker networks, cross-container communication, Postman |
| 04 | [Multi-Container App — Goals App](./4.multi-container/) | MongoDB + Node/Express + React in separate containers, custom Docker networks, named/anonymous volumes, bind mounts for live code updates, Nodemon, environment variables for credentials, database authentication |
| 05 | [Docker Compose — Goals App](./5.docker-compose/) | `docker-compose.yaml`, `services`, `build` vs `image`, automatic default networks, named/anonymous volumes & bind mounts with relative paths, `env_file`, `depends_on`, `docker-compose up`/`down`, service names vs container names |
| 06 | [Utility Containers — Node/npm Utility](./6.utility-containers/) | Application containers vs Utility containers, `docker exec`, overriding the default `CMD`, `ENTRYPOINT` vs `CMD`, bind mounts for utility containers, `docker compose run` vs `docker compose up`, `--rm` cleanup |
> More projects will be added as I progress through the course. 🚀
---
## 🛠️ Core Docker Commands Learned So Far
```bash
docker build -t <image-name> .              # Build an image
docker build --build-arg PORT=8080 .        # Build with custom ARG
docker run -d -p 3000:80 <image>            # Run a container
docker run -v name:/path <image>            # Run with a named volume
docker run -v $(pwd):/app <image>           # Run with a bind mount
docker run -v /app/node_modules <image>     # Run with an anonymous volume (protects container's own folder)
docker run --env-file .env <image>          # Run with an env file
docker run -e KEY=value <image>             # Run with an inline environment variable
docker run --network <name> <image>         # Run a container attached to a specific network
docker run -it <image>                      # Run in interactive mode (e.g. React dev server)
docker ps                                   # List running containers
docker stop <container>                     # Stop a container
docker start <container>                    # Start a container
docker rm <container>                       # Remove a container
docker images                               # List all images
docker rmi <image>                          # Remove an image
docker logs <container>                     # View a container's logs
docker volume ls                            # List all volumes
docker volume rm <volume>                   # Remove a volume
docker network create <name>                # Create a Docker network
docker network ls                           # List all networks
docker container inspect <container>        # Inspect container details (IP, etc.)
docker container prune                      # Remove all stopped containers
docker image prune -a                       # Remove all unused images
docker exec -it <container> <command>       # Run an extra command inside an already-running container
docker run -it <image> <command>            # Override the image's default CMD with a custom command

# Docker Compose
docker-compose up                           # Build (if needed) and start all services
docker-compose up -d                        # Start all services in detached mode
docker-compose up --build                   # Force rebuild of images before starting
docker-compose build                        # Build only the custom images (no containers started)
docker-compose down                         # Stop and remove containers + default network (keeps volumes)
docker-compose down -v                      # Stop and remove containers + network + volumes
docker compose run <service> <command>      # Run a one-off command in a service (for utility containers)
docker compose run --rm <service> <command> # Same as above, but auto-remove the container when done
```

### 🧰 Utility Containers (Subject 06)
Not every container has to run a full application. A **Utility Container** only bundles the *environment/tools* you need (e.g. Node.js/npm) so you can run one-off commands — like `npm init` or `npm install` — without installing anything on your host machine.

```dockerfile
FROM node:14-alpine
WORKDIR /app
ENTRYPOINT ["npm"]
```

```bash
docker build -t mynpm .
docker run -it -v $(pwd):/app mynpm init      # runs "npm init" inside the container
docker run -it -v $(pwd):/app mynpm install   # runs "npm install"
```

- `CMD` gets **overridden** by a command passed after the image name.
- `ENTRYPOINT` gets that command **appended as an argument** instead — great for building single-purpose "wrapper" images.
- Combine with a bind mount (`-v`) so files created inside the container (like `package.json`) show up on your host project folder.
- With Docker Compose, use `docker compose run --rm <service> <command>` instead of `docker compose up`, since utility containers are meant to run once and exit, not stay alive.
---
## 🔗 Resources
- [Docker Official Docs](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)