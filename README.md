# Load Balancer Test Application

This repository contains a simple test setup for an NGINX load balancer using Docker Compose. The configuration demonstrates how to distribute traffic between two backend services (`app` and `app2`) and includes instructions to disable access logs in NGINX.

## Features

- **Load Balancing:** NGINX is configured to balance traffic between two backend services.
- **Dockerized Environment:** Both NGINX and backend services are containerized using Docker Compose.

> **Note:** This application is designed for testing purposes only and is not intended for production use.

## Project Structure

```
project/
├── docker-compose.yml
├── app/
│   └── Dockerfile
├── app2/
│   └── Dockerfile
└── nginx/
    ├── Dockerfile
    └── nginx.conf
```

## How It Works

1. **Backend Services:**
   - Two Nestjs aplications (`app` and `app2`) run on ports `3000` and `3001`, respectively.

2. **NGINX Configuration:**
   - NGINX acts as a load balancer for the two services.
   - Traffic is distributed using the `proxy_pass` directive to the `proxy` upstream block.

## Usage

### Prerequisites

- Docker

### Steps to Run

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
   - Traffic will be distributed between `app` and `app2`.
   - When accessed through the load balancer, these services will return:
        - app: Responds with Hello World!
        - app2: Responds with Hello World!2

### Stop the Services

To stop and clean up the services, run:
```bash
docker-compose down
```

## Configuration Details

### NGINX Configuration (`nginx/nginx.conf`)

```nginx
upstream proxy {
    server app:3000;
    server app2:3001;
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
  app:
    build: "./app"
    ports:
      - "3000:3000"
    networks:
      - webnet

  app2:
    build: "./app2"
    ports:
      - "3001:3001"
    networks:
      - webnet

  nginx:
    build: "./nginx"
    ports:
      - "80:80"
    depends_on:
      - app
      - app2
    networks:
      - webnet

networks:
  webnet:
```

## License

This project is distributed under the MIT License.