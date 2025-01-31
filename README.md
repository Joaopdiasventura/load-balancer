# Load Balancer Application

This repository contains a setup for an NGINX load balancer using Docker. The configuration demonstrates how to distribute traffic between two services.

## Features

- **Load Balancing:** NGINX is configured to balance traffic between two services.
- **Dockerized Environment:** Both NGINX and services are containerized using Docker Compose.

## How It Works

1. **Services:**

   - Two Nestjs services (`app-1` and `app-2`) run on ports `3000` and `3001`, respectively.

2. **NGINX Configuration:**
   - NGINX acts as a load balancer for the two services.
   - Traffic is distributed using the `round-robin` `(rotation)` strategy.

## Configuration Details

### NGINX Configuration (`nginx/nginx.conf`)

```nginx
upstream proxy {
    server ${SERVER_1};
    server ${SERVER_2};
}

server {
    listen 80;
    access_log off;

    location / {
        proxy_pass http://proxy;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

```

### Docker Compose (`compose.yaml`)

The `compose.yaml` file sets up the services and ensures they are connected to the same network:

```yaml
services:
  app-1:
    build: "./app-1"
    ports:
      - "3000:3000"
    networks:
      - webnet

  app-2:
    build: "./app-2"
    ports:
      - "3001:3001"
    networks:
      - webnet

  nginx:
    build: "./nginx"
    environment:
      - SERVER_1=app-1:3000
      - SERVER_2=app-2:3001
    ports:
      - "80:80"
    depends_on:
      - app-1
      - app-2
    networks:
      - webnet

networks:
  webnet:
```

## Usage

### Prerequisites

- Docker

### Steps to Test

1. Clone this repository:

   ```bash
   git clone https://github.com/Joaopdiasventura/load-balancer.git
   cd load-balancer
   ```

2. Build and start the services:

   ```bash
   docker-compose up --build
   ```

3. Access the load balancer:
   - Open your browser or use `curl` to visit `http://localhost`.
   - Traffic will be distributed between `app-1` and `app-2`.
   - When accessed through the load balancer, these services will return:
     - app-1: Responds with Hello World!
     - app-2: Responds with Hello World!2

### Running the Load Balancer for your services

Instead of building the image locally, you can pull the pre-built image directly from **Docker Hub** and use it to balance your services:

1. **Pull the Docker image:**

   ```bash
   docker pull jpplay/nginx:latest
   ```

2. **Run the container:**

   ```bash
   docker run -d -p 80:80
     -e SERVER_1="app-1:port"
     -e SERVER_2="app-2:port"
     --name nginx jpplay/nginx:latest
   ```

> **Note: Make sure your services (`app-1:port` and `app-2:port`) are avaliable** before starting the container.
