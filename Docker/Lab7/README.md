# Lab 7: Multi-Network Microservices with Docker

## Objective

Deploy a three-tier microservices application using Docker containers, custom bridge networks, named volumes, and bind mounts.

## Application Architecture

The application consists of three Python Flask microservices:

| Service      | Container Port | Docker Network       | Purpose                       |
| ------------ | -------------- | -------------------- | ----------------------------- |
| Frontend     | 3000           | frontend-backend     | Web user interface            |
| Backend      | 5000           | Both networks        | Processes and stores messages |
| Notification | 5001           | backend-notification | Handles message notifications |

**Communication Flow:**

```text
Browser (localhost:8080)
          |
          v
     Frontend:3000
          |
    frontend-backend
     172.25.0.0/24
          |
          v
      Backend:5000
          |
   backend-notification
     172.26.0.0/24
          |
          v
    Notification:5001
```

The backend connects to both networks, while the frontend and notification services are isolated from one another.

## Technologies Used

- Docker and Docker CLI
- Python 3.12
- Flask
- Docker Bridge Networks
- Docker Named Volumes
- Docker Bind Mounts
- Windows PowerShell

## Step 1: Clone the Repository

```bash
git clone https://github.com/Ibrahim-Adel15/MicroserviceApp.git
cd MicroserviceApp
```

## Step 2: Build Docker Images

Each microservice has its own Dockerfile.

```bash
docker build -t lab7-backend ./backend
docker build -t lab7-frontend ./frontend
docker build -t lab7-notification ./notification
```

Verify the images:

```bash
docker images
```

## Step 3: Create Docker Networks

Create two custom bridge networks with separate subnets:

```bash
docker network create --driver bridge --subnet 172.25.0.0/24 frontend-backend
docker network create --driver bridge --subnet 172.26.0.0/24 backend-notification
```

Verify:

```bash
docker network ls
docker network inspect frontend-backend
docker network inspect backend-notification
```

## Step 4: Configure Persistent Storage

Create two named volumes:

```bash
docker volume create backend-logs
docker volume create notification-logs
```

Create a host directory for backend application data:

```powershell
mkdir backend-data
```

Storage configuration:

| Storage           | Container Path | Type         |
| ----------------- | -------------- | ------------ |
| backend-logs      | /app/logs      | Named Volume |
| notification-logs | /app/logs      | Named Volume |
| backend-data      | /app/data      | Bind Mount   |

Backend messages are stored in `/app/data/messages.json`.

## Step 5: Run the Notification Service

```powershell
docker run -d `
  --name notification `
  --network backend-notification `
  -v notification-logs:/app/logs `
  lab7-notification
```

Verify:

```bash
docker logs notification
```

The notification service listens on port 5001 within the Docker network.

## Step 6: Run the Backend Service

Run this command from the project root in PowerShell:

```powershell
docker run -d `
  --name backend `
  --network backend-notification `
  -e NOTIFICATION_URL=http://notification:5001/notify `
  -v backend-logs:/app/logs `
  --mount "type=bind,source=$((Get-Location).Path)\backend-data,target=/app/data" `
  lab7-backend
```

Connect the backend to the second network:

```bash
docker network connect frontend-backend backend
```

The backend now communicates with both the frontend and notification services.

## Step 7: Run the Frontend Service

```powershell
docker run -d `
  --name frontend `
  --network frontend-backend `
  -p 8080:3000 `
  lab7-frontend
```

Access the application at:

**http://localhost:8080**

## Step 8: Verification Commands

Check running containers:

```bash
docker ps
```

Inspect the networks:

```bash
docker network inspect frontend-backend
docker network inspect backend-notification
```

Check service logs:

```bash
docker logs frontend
docker logs backend
docker logs notification
```

Check backend health:

```powershell
docker exec backend python -c "import requests; print(requests.get('http://localhost:5000/health').json())"
```

Check persistent messages after submitting data through the application:

```powershell
Get-Content .\backend-data\messages.json
```

## Key Concepts

**Docker Bridge Networks:** Provide container-to-container communication and logical network separation.

**Multi-Network Containers:** Allow the backend to communicate with services on two separate networks.

**Docker Named Volumes:** Store application logs independently of the container lifecycle.

**Bind Mounts:** Map a host directory into a container so application data is stored on the host.

**Docker DNS:** Allows services to communicate using container names, such as `notification`, rather than fixed IP addresses.

**Port Mapping:** Exposes the frontend's internal port 3000 on host port 8080.

## Conclusion

This lab demonstrates the deployment of three containerized microservices using isolated Docker networks and persistent storage.

The frontend communicates with the backend, while the backend connects to the notification service through a separate network. Named volumes are configured for logging, and a bind mount is configured for backend application data.

The application is accessible at `http://localhost:8080`.

---

-
