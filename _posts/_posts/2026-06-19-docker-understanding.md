---
title: "Docker : to understand, to install and to use"
date: 2026-06-19 10:00:00 +0200
categories: [Tools]
tags: [Docker, DevOps]
---


# Docker

Imagine an app that runs on your machine but not on others. Docker tries to prevent this by running apps inside isolated containers. It makes development and deployment easier. Each container includes its own system, libraries, and dependencies, so your machine stays clean and everything works the same everywhere.

## How does it work?

Docker includes important concepts: **Images, Dockefiles** and **Containers**.

Docker images contain the technologies we need, the runtimes, and the tools/instructions to run the code. They simplify the process of sharing code. 

A Dockerfile is the recipe used to build an image. It contains step‑by‑step instructions (install dependencies, copy files, run commands) that Docker uses to assemble the final image.

Containers are like mini‑computers that include everything an app needs to run. Containers are:

- **self‑contained** → they contain all dependencies required to run an app
- **isolated** → each container runs in its own environment, avoiding conflicts
- **independent and portable** → they can run everywhere, on any computer

## What is ‘Docker Compose’?

Docker Compose is a tool for running multi‑container Docker applications. It lets you configure all your app’s services and networks in a single YAML file. Compose is a declarative tool.

How does it work?

1. Declare the app stack in a YAML file (`compose.yml`) and specify images, ports, volumes (volumes are persistent storage used by containers. Unlike the container’s filesystem, volumes are not deleted when the container stops. They are used for databases, logs, or any data you want to keep between runs), networks (Docker networks allow containers to communicate with each other securely and predictably. Services in the same Compose project automatically share a private network.), environment variables, and dependencies.
2. Navigate to the project directory containing the YAML file.
3. Start your app stack:

```bash
docker compose up -d --build
## it creates networks, volumes and starts all containers as defined in the yml file.
#-d means detached mode: containers run in the background so your terminal stays free.
```

Docker Compose commands can be used to stop, start, rebuild, and remove containers. Here are the main commands to manage multi‑container setups:

```bash
docker compose up        # Create and start containers
docker compose down      # Stop and remove containers + networks
docker compose build     # Build or rebuild services
docker compose start     # Start services
docker compose stop      # Stop services
docker compose restart   # Restart service containers
docker compose logs      # View output from containers
docker compose ps        # List containers and their status
docker compose exec      # Execute a command in a running container
docker compose run       # Run a one-off command on a service
docker compose pull      # Pull service images
docker compose push      # Push service images
docker compose rm        # Remove stopped service containers
docker compose images    # List images used by the created containers
docker compose ls        # List running Compose projects
docker compose pause     # Pause services
docker compose unpause   # Unpause services
docker compose top       # Display running processes
docker compose stats     # Live resource usage statistics
docker compose config    # Parse and render the Compose file in canonical format
docker compose version   # Show Docker Compose version information
docker compose volumes   # List volumes

```

You can also scale and update services easily by modifying the YAML file.

## What is Docker build?

`docker build` is a core Docker feature used to create container images from a Dockerfile and a build context. When you run the command, Docker reads the instructions in the Dockerfile, pulls any necessary base images, and executes each step to assemble a new image.

It packages your app and dependencies into an image. The build context is the directory (or URL) of the project.

```bash
docker build .
# . refers to the current directory
#The build process uses Buildx (the client) and BuildKit (the builder server) under the hood, both of which are included with Docker Desktop and Docker Engine.
#The resulting image can be run as a container, shared, or published to a registry.
```

## How to install docker?

**What I did (to install my blog website using docker)**

If you are in windows and use WSL, you can installed Docker Desktop :[https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/) 

Then ensure WSL2 (Linux for windows) integration. 

```bash
wsl --version
#expected output: WSL version: 2.x.x
#otherwise : wsl --install
```

Check that a Linux distro (e.g., Ubuntu) is installed:

```bash
wsl --list --verbose
#expected output: Ubuntu        Running        2
```

Enable the WSL2 backend in Docker Desktop:
**Docker Desktop → Settings → General**

Install the **Dev Containers** (Dev Containers let you develop *inside* the same environment that runs your app, ensuring perfect consistency between development and production) extension in VS Code.

Clone your GitHub repo directly into a Dev Container:

```bash
git clone <your-repo-url>
```

Build and Start the Docker App Stack

```bash
docker compose up -d --build
#--build forces Docker to rebuild images
#-d runs everything in the background
#Docker will start all services (web, db, cache..)
```

**Check That Containers Are Running**

```bash
docker ps
#status should be up
```

Access to app by opening your browser and go to the URL defined in your compose file. 

Stop the environment:

```bash
docker compose down
```

Fixed SSL certificate issues inside the container

Inside a container, you are running a mini‑Linux system. Many base images (Ruby, Python, Node, etc.) don’t include CA certificates by default. So when a tool like Bundler tries to access:

`https://rubygems.org/`

it cannot verify the SSL certificate.

Fix:

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates
sudo update-ca-certificates
bundle install
```

Once installed, HTTPS connections work normally and tools like Bundler can fetch dependencies securely.

Successfully launched the site using

```bash
bundle exec jekyll serve
#It builds the site ad launches a server to preview it (specific to my project)
```

**Result** 

Docker gives you reproducibility, isolation, portability, and a clean development environment. It removes the friction of installing dependencies manually and ensures your app behaves the same on every machine.

## Source

[https://docs.docker.com/build/](https://docs.docker.com/build/)

https://docs.docker.com/get-started/docker-concepts/building-images/build-tag-and-publish-an-image/

[https://docs.docker.com/desktop/](https://docs.docker.com/desktop/)

[What is Docker?](https://docs.docker.com/get-started/docker-overview/)

[https://docs.docker.com/reference/cli/docker/compose/](https://docs.docker.com/reference/cli/docker/compose/)

[https://www.youtube.com/watch?v=DQdB7wFEygo&t=243s](https://www.youtube.com/watch?v=DQdB7wFEygo&t=243s)
