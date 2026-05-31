# 🐳 Node.js Express App — Dockerized

A simple Node.js/Express web app where you can set and display a personal course goal. This project is part of my Docker learning journey — I took an existing app and containerized it by writing a `Dockerfile`, building an image, and running it as a container.

---

## 📦 Tech Stack

- **Runtime:** Node.js 14
- **Framework:** Express.js
- **Extras:** body-parser
- **Containerization:** Docker

---

## 🖥️ What the App Does

A minimal web page where you can:
- View your current **course goal**
- Submit a new goal via a form
- The default goal is: *"Learn Docker!"*

---

## 🐋 Docker Concepts Practiced

- Writing a `Dockerfile` from scratch
- Building a Docker **image** using `docker build`
- Running a Docker **container** using `docker run`
- Understanding the difference between **images** (blueprints) and **containers** (running instances)
- Using Docker flags: detached mode (`-d`), port mapping (`-p`), naming (`--name`)
- Managing containers and images via the Docker CLI

---

## 🚀 Getting Started

### Prerequisites

- [Docker](https://docs.docker.com/get-docker/) installed on your machine

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/your-repo-name.git
cd your-repo-name
```

### 2. Build the Docker Image

```bash
docker build -t course-goal-app .
```

### 3. Run the Container

```bash
docker run -d -p 3000:80 --name goal-container course-goal-app
```

Then open your browser and visit: **http://localhost:3000**

> The app runs on **port 80** inside the container, mapped to **port 3000** on your machine.

---

## 🛠️ Useful Docker Commands

| Command | Description |
|---|---|
| `docker build -t course-goal-app .` | Build the image |
| `docker run -d -p 3000:80 --name goal-container course-goal-app` | Run the container |
| `docker ps` | List running containers |
| `docker stop goal-container` | Stop the container |
| `docker start goal-container` | Start it again |
| `docker rm goal-container` | Remove the container |
| `docker images` | List all images |
| `docker rmi course-goal-app` | Remove the image |

---

## 📁 Project Structure

```
.
├── Dockerfile
├── package.json
├── server.js
└── public/
    └── styles.css
```

---

## 📄 Dockerfile

```dockerfile
FROM node:14

WORKDIR /app

COPY package.json /app

RUN npm install

COPY . /app

EXPOSE 80

CMD ["node", "server.js"]
```

**Why this order?** `package.json` is copied and `npm install` runs *before* copying the rest of the source code. This way Docker can **cache the dependencies layer** — if only `server.js` changes, Docker skips reinstalling packages and the build is much faster.

---

## 📚 What I Learned

- Docker separates **building** (image) from **running** (container)
- Images are built in **layers** — Docker caches unchanged layers for faster rebuilds
- You can run **multiple containers** from the same image, each fully isolated
- Port mapping (`-p host:container`) is needed to access the app from your browser
- Docker Hub can be used to share images with `docker push` / `docker pull`

---

## 🔗 Resources

- [Docker Official Docs](https://docs.docker.com/)
- [Node.js on Docker Hub](https://hub.docker.com/_/node)
- [Docker Hub](https://hub.docker.com/)
