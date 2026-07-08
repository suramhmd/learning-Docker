# Docker Networking – SW Favorites API

A Node.js REST API that lets you save your favorite Star Wars movies and characters.
Built as a hands-on project to learn **Docker container networking**.

---

## What This Project Covers

This project explores the three types of communication a Dockerized container can have:

| Type | Description | How |
|------|-------------|-----|
| Container → Internet | Fetch data from an external API | Works out of the box |
| Container → Host Machine | Connect to a service running locally on your machine | Use `host.docker.internal` |
| Container → Container | Connect to another container (e.g. a database) | Use a Docker Network |

---

## Tech Stack

- **Node.js** + **Express** – REST API server
- **MongoDB** – Database (run as a separate container)
- **Mongoose** – MongoDB connection & modeling
- **Axios** – HTTP requests to the Star Wars API ([swapi.py4e.com](https://swapi.py4e.com))
- **Docker** – Containerization & networking
- **Postman** – Sending HTTP requests to test the API

---

## Testing with Postman

All endpoints were tested using [Postman](https://www.postman.com). Example requests:

```
GET  http://localhost:3000/movies
GET  http://localhost:3000/people
GET  http://localhost:3000/favorites
POST http://localhost:3000/favorites
```

For the POST request, set the body to **raw / JSON**:

```json
{
  "name": "A New Hope",
  "type": "movie",
  "url": "https://swapi.dev/api/films/1/"
}
```

---

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/movies` | Fetch all Star Wars movies (from SWAPI) |
| `GET` | `/people` | Fetch all Star Wars characters (from SWAPI) |
| `GET` | `/favorites` | Get all saved favorites from the database |
| `POST` | `/favorites` | Save a new favorite |

### POST `/favorites` – Request Body

```json
{
  "name": "A New Hope",
  "type": "movie",
  "url": "https://swapi.dev/api/films/1/"
}
```

> `type` must be either `"movie"` or `"character"`.

---

## Running the Project

### 1. Create a Docker Network

Docker does not create networks automatically, so we create one first:

```bash
docker network create favorites-net
```

### 2. Run the MongoDB Container

```bash
docker run -d --name mongodb --network favorites-net mongo
```

> No `-p` flag needed here — only the Node container needs to reach MongoDB,
> and they're on the same network.

### 3. Build the Node.js Image

```bash
docker build -t favorites-node .
```

### 4. Run the Node.js Container

```bash
docker run --name favorites -d --rm -p 3000:3000 --network favorites-net favorites-node
```

The API is now available at `http://localhost:3000`.

---

## Key Docker Networking Concepts Learned

### Container → Internet
Containers can send HTTP requests to external websites and APIs with **no extra configuration**. This works out of the box.

### Container → Host Machine
Using `localhost` inside a container does **not** point to your host machine.
Instead, use the special domain:

```
host.docker.internal
```

Docker automatically resolves this to the IP address of your host machine.

### Container → Container (Docker Networks)
Placing containers in the same Docker Network allows them to communicate using **container names as hostnames**:

```js
// Instead of hardcoding an IP like 172.17.0.2:27017
mongoose.connect('mongodb://mongodb:27017/swfavorites')
//                            ↑ container name
```

Docker resolves the container name to its internal IP at request time — no code changes needed when IPs change.

### Port Publishing Rules

```
# Container ↔ Container (same network) → no -p needed
docker run --name mongodb --network favorites-net mongo

# Host Machine ↔ Container → -p required
docker run -p 3000:3000 --network favorites-net favorites-node
```

---

## Project Structure

```
.
├── app.js          # Express server & route handlers
├── package.json    # Node dependencies
├── Dockerfile      # Image definition for the Node app
└── models/
    └── favorite.js # Mongoose schema
```

---

## Notes

- Data stored in the MongoDB container **will be lost** when the container stops.
  This can be fixed by adding a **Docker Volume** to the MongoDB container (covered in the next module).
- The Node container uses `--rm` so it's automatically removed when stopped.