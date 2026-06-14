🐳 Learning Docker — From Zero to Confident
A hands-on collection of projects built throughout my Docker learning journey.
Each folder represents a new concept or milestone — starting from the basics and growing into more complex, real-world use cases.

📖 About This Course
This repo follows a structured Docker course that covers everything from the ground up — no prior Docker experience required.
Throughout the course I'm learning:
What Docker is and the core concepts behind it
What Containers are and how they work
The most important Docker commands and what you can do with them
How to use Docker locally for both simple and complex projects
How to deploy applications using Docker
How to solve real-world problems with Docker
The course is heavy on practical examples and demos, not just theory.
By the end, the goal is to feel confident using Docker in any new project.

📁 Projects
#	Project	Concepts Covered
01	First Container — Node.js App	Dockerfile, docker build, docker run, images vs containers, port mapping
02	Data & Volumes — Feedback App	Named volumes, anonymous volumes, bind mounts, ENV, ARG, .dockerignore

More projects will be added as I progress through the course. 🚀

🛠️ Core Docker Commands Learned So Far
docker build -t <image-name> .              # Build an image
docker build --build-arg PORT=8080 .        # Build with custom ARG
docker run -d -p 3000:80 <image>            # Run a container
docker run -v name:/path <image>            # Run with a named volume
docker run -v $(pwd):/app <image>           # Run with a bind mount
docker run --env-file .env <image>          # Run with an env file
docker ps                                   # List running containers
docker stop <container>                     # Stop a container
docker start <container>                    # Start a container
docker rm <container>                       # Remove a container
docker images                               # List all images
docker rmi <image>                          # Remove an image
docker volume ls                            # List all volumes
docker volume rm <volume>                   # Remove a volume

🔗 Resources
Docker Official Docs
Docker Hub
