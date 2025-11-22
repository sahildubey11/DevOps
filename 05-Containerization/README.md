# Containerization with Docker

## What is Containerization?

Containerization is a lightweight form of virtualization that packages an application and its dependencies together in an isolated environment called a container. Containers share the host OS kernel but run as isolated processes.

## Containers vs Virtual Machines

### Virtual Machines
```
┌─────────────────────────────────┐
│     App A     │     App B       │
│  Bins/Libs    │   Bins/Libs     │
├───────────────┼─────────────────┤
│   Guest OS    │   Guest OS      │
├───────────────┴─────────────────┤
│         Hypervisor              │
├─────────────────────────────────┤
│         Host OS                 │
├─────────────────────────────────┤
│      Infrastructure             │
└─────────────────────────────────┘
```

### Containers
```
┌─────────────────────────────────┐
│  App A  │  App B  │  App C      │
│ Bins/   │ Bins/   │ Bins/       │
│ Libs    │ Libs    │ Libs        │
├─────────┴─────────┴─────────────┤
│    Container Runtime (Docker)   │
├─────────────────────────────────┤
│          Host OS                │
├─────────────────────────────────┤
│      Infrastructure             │
└─────────────────────────────────┘
```

### Key Differences

| Aspect | Virtual Machines | Containers |
|--------|-----------------|------------|
| Size | GBs | MBs |
| Startup | Minutes | Seconds |
| OS | Separate OS per VM | Shared host OS |
| Isolation | Complete isolation | Process-level isolation |
| Performance | Lower (overhead) | Near-native |
| Resource Usage | Heavy | Lightweight |

## Benefits of Containers

1. **Portability**: Run anywhere - dev, test, prod
2. **Consistency**: Same environment everywhere
3. **Efficiency**: Lightweight, fast startup
4. **Isolation**: Dependencies don't conflict
5. **Scalability**: Easy to scale up/down
6. **Version Control**: Images are versioned
7. **Microservices**: Perfect for microservices architecture

## Docker Architecture

### Components

```
┌──────────────────────────────────────┐
│         Docker Client (CLI)          │
│    $ docker run, build, push, etc.   │
└──────────────┬───────────────────────┘
               │ REST API
┌──────────────▼───────────────────────┐
│         Docker Daemon (dockerd)      │
│  - Manages images, containers, etc.  │
│  - Builds, runs, distributes         │
└──────────────┬───────────────────────┘
               │
    ┌──────────┴──────────┬──────────┐
    │                     │          │
┌───▼────┐        ┌───────▼──┐  ┌───▼────┐
│ Images │        │Container │  │Networks│
└────────┘        └──────────┘  └────────┘
```

### Key Concepts

#### Images
- Read-only template for creating containers
- Built from Dockerfile
- Stored in registries (Docker Hub, ECR, etc.)
- Layered filesystem

#### Containers
- Running instance of an image
- Isolated process with own filesystem
- Can be started, stopped, deleted

#### Dockerfile
- Text file with instructions to build an image
- Each instruction creates a layer

#### Registry
- Repository for storing and distributing images
- Public: Docker Hub
- Private: AWS ECR, Google GCR, Azure ACR, Harbor

## Installing Docker

### Linux (Ubuntu)
```bash
# Update package index
sudo apt-get update

# Install prerequisites
sudo apt-get install ca-certificates curl gnupg

# Add Docker's official GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Set up repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add user to docker group (optional)
sudo usermod -aG docker $USER
```

### macOS
```bash
# Download Docker Desktop from docker.com
# Or use Homebrew
brew install --cask docker
```

### Windows
Download and install Docker Desktop from docker.com

## Docker Commands

### Images

```bash
# Pull image from registry
docker pull nginx:1.25
docker pull ubuntu:20.04

# List local images
docker images
docker image ls

# Build image from Dockerfile
docker build -t myapp:1.0 .
docker build -t myapp:latest -f Dockerfile.prod .

# Remove image
docker rmi nginx:1.25
docker image rm myapp:1.0

# Tag image
docker tag myapp:1.0 myusername/myapp:1.0

# Push image to registry
docker push myusername/myapp:1.0

# Image history
docker history nginx:1.25

# Inspect image
docker inspect nginx:1.25
```

### Containers

```bash
# Run container
docker run nginx
docker run -d nginx                          # Detached mode
docker run -d -p 8080:80 nginx              # Port mapping
docker run -d --name webserver nginx        # Named container
docker run -it ubuntu bash                  # Interactive mode
docker run -d -v /host/path:/container/path nginx  # Volume mount

# List running containers
docker ps
docker container ls

# List all containers (including stopped)
docker ps -a
docker container ls -a

# Stop container
docker stop webserver
docker stop <container-id>

# Start stopped container
docker start webserver

# Restart container
docker restart webserver

# Remove container
docker rm webserver
docker rm -f webserver                      # Force remove running container

# View logs
docker logs webserver
docker logs -f webserver                    # Follow logs
docker logs --tail 100 webserver           # Last 100 lines

# Execute command in running container
docker exec -it webserver bash
docker exec webserver ls /etc

# View container resource usage
docker stats
docker stats webserver

# Inspect container
docker inspect webserver

# Copy files to/from container
docker cp file.txt webserver:/app/
docker cp webserver:/app/log.txt ./
```

### Cleanup

```bash
# Remove all stopped containers
docker container prune

# Remove all unused images
docker image prune
docker image prune -a                       # Remove all unused images

# Remove all unused volumes
docker volume prune

# Remove all unused networks
docker network prune

# Remove everything unused
docker system prune
docker system prune -a --volumes           # Remove everything
```

## Dockerfile

### Basic Structure

```dockerfile
# Syntax: INSTRUCTION arguments

# Base image
FROM ubuntu:20.04

# Metadata
LABEL maintainer="your-email@example.com"
LABEL version="1.0"

# Set working directory
WORKDIR /app

# Copy files
COPY package.json .
COPY . .

# Run commands during build
RUN apt-get update && \
    apt-get install -y python3 && \
    rm -rf /var/lib/apt/lists/*

# Environment variables
ENV NODE_ENV=production
ENV PORT=3000

# Expose ports
EXPOSE 3000

# Default command
CMD ["python3", "app.py"]
```

### Instructions

#### FROM
```dockerfile
FROM node:18-alpine
FROM python:3.9-slim
FROM ubuntu:20.04
```

#### RUN
```dockerfile
RUN apt-get update && apt-get install -y curl
RUN npm install
RUN pip install -r requirements.txt
```

#### COPY vs ADD
```dockerfile
# COPY - Simple copy
COPY app.py /app/
COPY . /app/

# ADD - Copy + extract tar + download URLs
ADD archive.tar.gz /app/
ADD https://example.com/file.txt /app/
```

#### CMD vs ENTRYPOINT
```dockerfile
# CMD - Default command (can be overridden)
CMD ["python3", "app.py"]
CMD python3 app.py

# ENTRYPOINT - Main command (not easily overridden)
ENTRYPOINT ["python3"]
CMD ["app.py"]  # Default argument to ENTRYPOINT

# Combined: docker run image arg.py
# Runs: python3 arg.py
```

#### WORKDIR
```dockerfile
WORKDIR /app
# All subsequent commands run from /app
```

#### ENV
```dockerfile
ENV NODE_ENV=production
ENV PATH="/app/bin:${PATH}"
```

#### EXPOSE
```dockerfile
EXPOSE 80
EXPOSE 443
EXPOSE 3000
```

#### VOLUME
```dockerfile
VOLUME /data
VOLUME ["/var/log", "/var/db"]
```

#### ARG
```dockerfile
ARG VERSION=latest
FROM node:${VERSION}

ARG BUILD_DATE
LABEL build-date=${BUILD_DATE}
```

### Multi-Stage Builds

Reduce image size by using multiple stages:

```dockerfile
# Build stage
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

### Example Dockerfiles

#### Node.js Application
```dockerfile
FROM node:18-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Expose port
EXPOSE 3000

# Set user (security)
USER node

# Start application
CMD ["node", "server.js"]
```

#### Python Application
```dockerfile
FROM python:3.9-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Create non-root user
RUN useradd -m appuser
USER appuser

EXPOSE 8000

CMD ["python", "app.py"]
```

#### Go Application
```dockerfile
# Build stage
FROM golang:1.19-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

# Production stage
FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/main .
EXPOSE 8080
CMD ["./main"]
```

## Docker Compose

Manage multi-container applications.

### Installation
```bash
# Linux
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Verify
docker-compose --version
```

### docker-compose.yml

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://db:5432/myapp
    depends_on:
      - db
      - redis
    volumes:
      - ./app:/app
      - /app/node_modules
    networks:
      - app-network
    restart: unless-stopped

  db:
    image: postgres:14-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    networks:
      - app-network

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - web
    networks:
      - app-network

volumes:
  db-data:

networks:
  app-network:
    driver: bridge
```

### Docker Compose Commands

```bash
# Start services
docker-compose up
docker-compose up -d                        # Detached mode
docker-compose up --build                   # Rebuild images

# Stop services
docker-compose stop
docker-compose down                         # Stop and remove containers
docker-compose down -v                      # Also remove volumes

# View logs
docker-compose logs
docker-compose logs -f web                  # Follow logs for web service

# List services
docker-compose ps

# Execute command in service
docker-compose exec web bash
docker-compose exec db psql -U user myapp

# Scale services
docker-compose up -d --scale web=3

# Restart service
docker-compose restart web

# Build images
docker-compose build
docker-compose build --no-cache
```

## Docker Networking

### Network Drivers

1. **bridge** (default): Isolated network on host
2. **host**: Share host's network
3. **none**: No network
4. **overlay**: Multi-host networking (Swarm)
5. **macvlan**: Assign MAC address to container

### Network Commands

```bash
# List networks
docker network ls

# Create network
docker network create mynetwork
docker network create --driver bridge mynetwork

# Connect container to network
docker network connect mynetwork container1

# Disconnect
docker network disconnect mynetwork container1

# Inspect network
docker network inspect mynetwork

# Remove network
docker network rm mynetwork
```

## Docker Volumes

### Types of Mounts

1. **Volumes**: Managed by Docker
2. **Bind Mounts**: Mount host directory
3. **tmpfs**: Temporary in memory

### Volume Commands

```bash
# Create volume
docker volume create myvolume

# List volumes
docker volume ls

# Inspect volume
docker volume inspect myvolume

# Remove volume
docker volume rm myvolume

# Remove unused volumes
docker volume prune

# Use volume in container
docker run -v myvolume:/data nginx
docker run --mount source=myvolume,target=/data nginx
```

### Bind Mounts

```bash
# Mount host directory
docker run -v /host/path:/container/path nginx
docker run -v $(pwd):/app node:18

# Read-only mount
docker run -v /host/path:/container/path:ro nginx
```

## Docker Best Practices

### 1. Use Official Images
```dockerfile
FROM node:18-alpine
FROM python:3.9-slim
```

### 2. Minimize Layers
```dockerfile
# Bad
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y vim

# Good
RUN apt-get update && \
    apt-get install -y curl vim && \
    rm -rf /var/lib/apt/lists/*
```

### 3. Use .dockerignore
```.dockerignore
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.DS_Store
dist
coverage
```

### 4. Don't Run as Root
```dockerfile
RUN useradd -m appuser
USER appuser
```

### 5. Use Multi-Stage Builds
Reduce final image size by separating build and runtime.

### 6. Scan for Vulnerabilities
```bash
docker scan myimage:latest
```

### 7. Pin Versions
```dockerfile
# Bad
FROM node:latest

# Good
FROM node:18.16.0-alpine3.17
```

### 8. Use Health Checks
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost/ || exit 1
```

### 9. Leverage Build Cache
- Order instructions from least to most frequently changed
- Copy dependency files first

```dockerfile
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
```

## Container Security

### Best Practices

1. **Use Minimal Base Images**: alpine, slim, distroless
2. **Don't Run as Root**: Create and use non-root user
3. **Scan Images**: Use Docker Scan, Trivy, Clair
4. **Sign Images**: Use Docker Content Trust
5. **Limit Resources**: Set CPU and memory limits
6. **Read-Only Filesystem**: When possible
7. **Drop Capabilities**: Reduce container privileges
8. **Network Segmentation**: Use networks appropriately
9. **Secrets Management**: Don't hardcode secrets
10. **Keep Updated**: Regularly update base images

### Security Commands

```bash
# Run with limited resources
docker run -m 512m --cpus 1 nginx

# Read-only root filesystem
docker run --read-only nginx

# Drop capabilities
docker run --cap-drop ALL --cap-add NET_BIND_SERVICE nginx

# Run as specific user
docker run --user 1000:1000 nginx

# Security scanning
docker scan nginx:1.25
```

## Resources

- Docker Documentation: https://docs.docker.com/
- Docker Hub: https://hub.docker.com/
- Dockerfile Best Practices: https://docs.docker.com/develop/dev-best-practices/
- Play with Docker: https://labs.play-with-docker.com/

## Next Steps

Continue to [Container Orchestration (Kubernetes)](../06-Container-Orchestration/) to learn about managing containers at scale.
