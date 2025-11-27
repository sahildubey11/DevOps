# Docker - Containerization

## What is Docker?

Docker is a platform for developing, shipping, and running applications in containers. Containers package an application with all its dependencies, ensuring it runs consistently across different environments.

## Why Docker?

- **Consistency**: "Works on my machine" → "Works everywhere"
- **Isolation**: Applications run in isolated environments
- **Portability**: Run anywhere Docker is installed
- **Efficiency**: Lightweight compared to VMs
- **Scalability**: Easy to scale up or down
- **Version Control**: Track container images like code

## Docker vs Virtual Machines

| Feature | Docker Containers | Virtual Machines |
|---------|------------------|------------------|
| Size | Lightweight (MBs) | Heavy (GBs) |
| Startup | Seconds | Minutes |
| Performance | Native | Overhead |
| Isolation | Process-level | Full OS |
| Portability | High | Moderate |

## Docker Architecture

### Components

1. **Docker Client**: CLI to interact with Docker
2. **Docker Daemon**: Background service managing containers
3. **Docker Images**: Read-only templates
4. **Docker Containers**: Running instances of images
5. **Docker Registry**: Repository for images (Docker Hub)

## Installation

### Ubuntu
```bash
# Update packages
sudo apt-get update

# Install dependencies
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common

# Add Docker's GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# Add Docker repository
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

# Install Docker
sudo apt-get update
sudo apt-get install docker-ce

# Verify installation
docker --version
```

### macOS
```bash
# Using Homebrew
brew install --cask docker
```

### Post-installation
```bash
# Add user to docker group (Linux)
sudo usermod -aG docker $USER

# Start Docker daemon
sudo systemctl start docker
sudo systemctl enable docker
```

## Essential Docker Commands

### Image Management
```bash
# Pull an image
docker pull nginx:latest

# List images
docker images

# Remove an image
docker rmi nginx:latest

# Build an image
docker build -t myapp:1.0 .

# Tag an image
docker tag myapp:1.0 myapp:latest

# Push to registry
docker push username/myapp:1.0

# Search for images
docker search nginx

# View image history
docker history nginx
```

### Container Management
```bash
# Run a container
docker run nginx

# Run container in detached mode
docker run -d nginx

# Run with port mapping
docker run -d -p 8080:80 nginx

# Run with name
docker run -d --name my-nginx nginx

# Run with environment variables
docker run -d -e "ENV=production" nginx

# Run with volume
docker run -d -v /host/path:/container/path nginx

# List running containers
docker ps

# List all containers
docker ps -a

# Stop a container
docker stop my-nginx

# Start a container
docker start my-nginx

# Restart a container
docker restart my-nginx

# Remove a container
docker rm my-nginx

# Remove running container (force)
docker rm -f my-nginx

# Execute command in running container
docker exec -it my-nginx bash

# View container logs
docker logs my-nginx

# Follow logs
docker logs -f my-nginx

# View container stats
docker stats

# Inspect container
docker inspect my-nginx

# Copy files to/from container
docker cp file.txt my-nginx:/path/
docker cp my-nginx:/path/file.txt ./
```

### System Commands
```bash
# Show Docker info
docker info

# Show disk usage
docker system df

# Clean up unused data
docker system prune

# Remove all stopped containers
docker container prune

# Remove all unused images
docker image prune

# Remove all unused volumes
docker volume prune
```

## Dockerfile

A Dockerfile contains instructions to build a Docker image.

### Basic Dockerfile Example (Node.js)

```dockerfile
# Base image
FROM node:16-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Expose port
EXPOSE 3000

# Set environment variable
ENV NODE_ENV=production

# Command to run application
CMD ["node", "server.js"]
```

### Multi-stage Build (Optimized)

```dockerfile
# Build stage
FROM node:16-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# Production stage
FROM node:16-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY --from=builder /app/dist ./dist

EXPOSE 3000

USER node

CMD ["node", "dist/server.js"]
```

### Python Application Dockerfile

```dockerfile
FROM python:3.9-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Create non-root user
RUN useradd -m appuser && chown -R appuser /app
USER appuser

EXPOSE 8000

CMD ["python", "app.py"]
```

### Dockerfile Instructions

| Instruction | Description |
|-------------|-------------|
| FROM | Base image |
| WORKDIR | Set working directory |
| COPY | Copy files from host to container |
| ADD | Like COPY but can extract archives |
| RUN | Execute commands during build |
| CMD | Default command to run container |
| ENTRYPOINT | Configure container as executable |
| EXPOSE | Document which ports are used |
| ENV | Set environment variables |
| ARG | Build-time variables |
| VOLUME | Create mount point |
| USER | Set user for RUN, CMD, ENTRYPOINT |
| LABEL | Add metadata to image |

## Docker Compose

Docker Compose allows you to define and run multi-container applications.

### docker-compose.yml Example

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DB_HOST=db
    depends_on:
      - db
      - redis
    volumes:
      - ./logs:/app/logs
    networks:
      - app-network

  db:
    image: postgres:14-alpine
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=admin
      - POSTGRES_PASSWORD=secret
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
      - ./nginx.conf:/etc/nginx/nginx.conf
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

# Start in detached mode
docker-compose up -d

# Stop services
docker-compose down

# Stop and remove volumes
docker-compose down -v

# View logs
docker-compose logs

# Follow logs
docker-compose logs -f

# List containers
docker-compose ps

# Execute command in service
docker-compose exec web bash

# Build images
docker-compose build

# Pull images
docker-compose pull

# Restart services
docker-compose restart

# Scale services
docker-compose up -d --scale web=3
```

## Docker Networking

### Network Types

1. **Bridge**: Default network, containers on same host
2. **Host**: Share host's network namespace
3. **None**: No network
4. **Overlay**: Multi-host networking
5. **Macvlan**: Assign MAC address to container

### Network Commands

```bash
# List networks
docker network ls

# Create network
docker network create my-network

# Inspect network
docker network inspect my-network

# Connect container to network
docker network connect my-network my-container

# Disconnect container from network
docker network disconnect my-network my-container

# Remove network
docker network rm my-network
```

## Docker Volumes

Volumes persist data outside container lifecycle.

### Volume Commands

```bash
# Create volume
docker volume create my-volume

# List volumes
docker volume ls

# Inspect volume
docker volume inspect my-volume

# Remove volume
docker volume rm my-volume

# Use volume in container
docker run -v my-volume:/data nginx
```

### Volume Types

1. **Named Volumes**: Managed by Docker
2. **Bind Mounts**: Mount host directory
3. **tmpfs Mounts**: Stored in host memory

```bash
# Named volume
docker run -v data-vol:/app/data nginx

# Bind mount
docker run -v /host/path:/container/path nginx

# Read-only volume
docker run -v /host/path:/container/path:ro nginx
```

## Docker Best Practices

### 1. Image Optimization

```dockerfile
# Use specific tags, not 'latest'
FROM node:16-alpine

# Minimize layers
RUN apt-get update && apt-get install -y \
    package1 \
    package2 \
    && rm -rf /var/lib/apt/lists/*

# Order layers by change frequency
COPY package.json .
RUN npm install
COPY . .

# Use .dockerignore
```

### 2. .dockerignore File

```
node_modules
npm-debug.log
Dockerfile
.dockerignore
.git
.gitignore
README.md
.env
.env.local
dist
coverage
.vscode
```

### 3. Security

```dockerfile
# Don't run as root
RUN useradd -m appuser
USER appuser

# Use minimal base images
FROM alpine:3.14

# Scan for vulnerabilities
docker scan myimage:latest

# Don't include secrets
ENV API_KEY=${API_KEY}  # Use build args
```

### 4. Multi-stage Builds

```dockerfile
# Build stage
FROM node:16 AS build
WORKDIR /app
COPY . .
RUN npm ci && npm run build

# Production stage
FROM node:16-alpine
WORKDIR /app
COPY --from=build /app/dist ./dist
RUN npm ci --only=production
CMD ["node", "dist/server.js"]
```

## Docker Registry

### Docker Hub

```bash
# Login
docker login

# Tag image
docker tag myapp:1.0 username/myapp:1.0

# Push image
docker push username/myapp:1.0

# Pull image
docker pull username/myapp:1.0
```

### Private Registry

```bash
# Run private registry
docker run -d -p 5000:5000 --name registry registry:2

# Tag for private registry
docker tag myapp:1.0 localhost:5000/myapp:1.0

# Push to private registry
docker push localhost:5000/myapp:1.0
```

## Docker in CI/CD

### Build and Push in Pipeline

```yaml
# GitHub Actions
- name: Build Docker image
  run: docker build -t myapp:${{ github.sha }} .

- name: Push to registry
  run: |
    echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
    docker push myapp:${{ github.sha }}
```

## Troubleshooting

### Common Issues

```bash
# Container exits immediately
docker logs container-name

# Check resource usage
docker stats

# Enter container for debugging
docker exec -it container-name sh

# View container processes
docker top container-name

# Check container events
docker events

# Inspect container details
docker inspect container-name
```

## Interview Questions

1. What is the difference between Docker and Virtual Machines?
2. Explain Docker architecture
3. What is the difference between CMD and ENTRYPOINT?
4. How do you optimize Docker images?
5. What is a multi-stage build?
6. Explain Docker networking types
7. What is the difference between COPY and ADD?
8. How do you persist data in Docker?
9. What is Docker Compose used for?
10. How do you handle secrets in Docker?

## Practical Exercises

1. Create a Dockerfile for a web application
2. Build and run a multi-container application with Docker Compose
3. Implement a multi-stage build to reduce image size
4. Set up a private Docker registry
5. Create a containerized development environment
6. Implement health checks in containers
7. Practice debugging running containers
8. Set up container monitoring

## Resources

- [Docker Documentation](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)
- [Dockerfile Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Play with Docker](https://labs.play-with-docker.com/)
- [Docker Cheat Sheet](https://www.docker.com/sites/default/files/d8/2019-09/docker-cheat-sheet.pdf)
