# Docker Setup and Usage

## Installation
1. Download Docker Desktop:
   - [Windows/Mac Download](https://www.docker.com/products/docker-desktop/)
   - For Linux, follow [Docker Engine installation](https://docs.docker.com/engine/install/)

2. Verify Installation:
   ```bash
   docker --version
   docker-compose --version

Test Docker:
bashCopydocker run hello-world


Essential Commands
Container Management
bashCopy# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Start a container
docker start <container_name>

# Stop a container
docker stop <container_name>

# Remove a container
docker rm <container_name>
Image Management
bashCopy# List images
docker images

# Pull an image
docker pull python:3.9

# Remove an image
docker rmi <image_name>

# Build an image
docker build -t myproject .
Project Setup
Basic Python Dockerfile Example
dockerfileCopyFROM python:3.9

# Set working directory
WORKDIR /app

# Copy and install requirements first (better caching)
COPY requirements.txt .
RUN pip install -r requirements.txt

# Copy project files
COPY . .

# Command to run the application
CMD ["python", "app.py"]
Docker Compose Example
yamlCopyversion: '3'
services:
  app:
    build: .
    volumes:
      - .:/app
    ports:
      - "8000:8000"
    environment:
      - DEBUG=1
Development Workflow

Create Dockerfile and docker-compose.yml
Build image: docker-compose build
Start services: docker-compose up
View logs: docker-compose logs
Stop services: docker-compose down

Best Practices

Use specific version tags for base images
Minimize number of layers
Use .dockerignore file
Don't run as root user
Clean up unused images and containers regularly

Debugging Tips

Access container shell: docker exec -it <container_name> bash
View container logs: docker logs <container_name>
Check container details: docker inspect <container_name>
