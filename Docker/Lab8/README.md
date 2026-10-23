# Lab 8: Containerized Node.js and MySQL Stack Using Docker Compose

## Objective

Containerize a Node.js application and MySQL database using Docker Compose, configure communication between services through a custom Docker network, and implement persistent storage for database files and application logs.

The lab also demonstrates how to build, tag, and publish a Docker image to Docker Hub.

## Technologies Used

- Docker and Docker Compose
- Node.js 18
- Express.js
- MySQL 8.0
- Morgan HTTP Logger
- Docker Named Volumes
- Docker Bridge Networking
- Docker Hub

## 1. Clone the Repository

```bash
git clone https://github.com/Ibrahim-Adel15/iVolveApp.git
cd iVolveApp
```

The application consists of a Node.js backend, a static frontend, and MySQL database connectivity.

The application exposes three HTTP endpoints:

| Endpoint  | Purpose                                      |
| --------- | -------------------------------------------- |
| `/`       | Serves the application frontend              |
| `/health` | Checks application and database connectivity |
| `/ready`  | Checks application readiness                 |

The Node.js application listens on port `3000`.

## 2. Dockerfile

The provided Dockerfile builds the Node.js application image.

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["node", "server.js"]
```

### Explanation

- `FROM`: Uses Node.js 18 on Alpine Linux.
- `WORKDIR`: Sets `/app` as the working directory.
- `COPY`: Copies application files into the image.
- `RUN npm install`: Installs Node.js dependencies.
- `EXPOSE`: Documents port 3000.
- `CMD`: Starts the Node.js server.

## 3. Docker Compose Configuration

Created `docker-compose.yml` to manage both the Node.js application and the MySQL database.

```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    image: ivolve-app:lab8
    container_name: ivolve-app
    ports:
      - "3000:3000"
    environment:
      DB_HOST: db
      DB_USER: root
      DB_PASSWORD: ${MYSQL_ROOT_PASSWORD}
    volumes:
      - app_logs:/app/logs
    depends_on:
      db:
        condition: service_healthy
    networks:
      - ivolve-net
    restart: unless-stopped

  db:
    image: mysql:8.0
    container_name: ivolve-mysql
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ivolve
    volumes:
      - db_data:/var/lib/mysql
    healthcheck:
      test:
        [
          "CMD-SHELL",
          'mysqladmin ping -h localhost -uroot -p"$$MYSQL_ROOT_PASSWORD" --silent',
        ]
      interval: 10s
      timeout: 5s
      retries: 10
      start_period: 30s
    networks:
      - ivolve-net
    restart: unless-stopped

networks:
  ivolve-net:
    name: ivolve-net
    driver: bridge

volumes:
  db_data:
  app_logs:
```

## 4. Environment Configuration

Created a `.env` file to store the MySQL root password.

```env
MYSQL_ROOT_PASSWORD=<your-local-mysql-password>
```

Docker Compose substitutes this value into the application and database service configurations.

The `.env` file is excluded from Git tracking because it may contain sensitive credentials.

Add the following to `.gitignore`:

```gitignore
.env
node_modules/
logs/
```

## 5. Docker Compose Architecture

```text
                 User / Browser
                       |
                       | HTTP :3000
                       |
               +----------------+
               |   Node.js App  |
               |    Express     |
               |   Port 3000    |
               +----------------+
                  |          |
                  |          |
          MySQL Connection   Access Logs
                  |          |
                  v          v
          +------------+  +-----------+
          |   MySQL    |  | app_logs  |
          |  Port 3306 |  |  Volume   |
          +------------+  +-----------+
                 |
                 v
          +------------+
          |  db_data   |
          |   Volume   |
          +------------+

       Custom Network: ivolve-net
```

Both containers communicate using the custom bridge network `ivolve-net`.

The Node.js application connects to MySQL using the service hostname `db`, which Docker Compose resolves through internal DNS.

MySQL port 3306 is not published to the host because database access is only required internally.

## 6. Build and Start the Application

Validate the Compose configuration:

```bash
docker compose config
```

Build and start both services:

```bash
docker compose up -d --build
```

Check running containers:

```bash
docker compose ps
```

The application and database containers were created successfully, and MySQL reached a healthy state.

## 7. Verify Application and Database Connectivity

Check application logs:

```bash
docker compose logs app --tail=30
```

Expected successful startup messages:

```text
Connected to MySQL and 'ivolve' DB found.
Server started on http://0.0.0.0:3000
```

Access the application at:

```text
http://localhost:3000
```

Test health and readiness:

```powershell
Invoke-WebRequest http://localhost:3000/health
Invoke-WebRequest http://localhost:3000/ready
```

Both endpoints return HTTP 200 when database connectivity is available.

## 8. Verify the MySQL Database

Access the MySQL container:

```bash
docker compose exec db mysql -uroot -p
```

Run:

```sql
SHOW DATABASES;
USE ivolve;
SHOW TABLES;
```

The `ivolve` database is initialized using the `MYSQL_DATABASE` environment variable.

The application checks for this database before starting its HTTP server.

## 9. Persistent Database Storage

A named Docker volume is attached to the MySQL data directory:

```yaml
volumes:
  - db_data:/var/lib/mysql
```

This allows MySQL database files to survive container removal and recreation.

To inspect volumes:

```bash
docker volume ls
```

Stop and remove the containers without removing volumes:

```bash
docker compose down
```

Recreate the services:

```bash
docker compose up -d
```

The Compose stack was successfully recreated without deleting its persistent volumes.

**Note:** `docker compose down -v` removes the named volumes and should be avoided when persistent data must be retained.

For a full persistence test, create a database record before restarting and verify that the same record exists afterward.

## 10. Persistent Application Logs

The Node.js application uses Morgan to write HTTP access logs to:

```text
/app/logs/access.log
```

Docker Compose mounts a named volume:

```yaml
volumes:
  - app_logs:/app/logs
```

Generate application traffic:

```powershell
Invoke-WebRequest http://localhost:3000/health
```

Inspect the access log:

```bash
docker compose exec app cat /app/logs/access.log
```

Because the log directory is backed by a Docker named volume, its contents can be preserved when the application container is recreated.

## 11. Custom Docker Network

Both services are connected to the custom bridge network:

```yaml
networks:
  ivolve-net:
    name: ivolve-net
    driver: bridge
```

Inspect the network:

```bash
docker network inspect ivolve-net
```

Verify that the Node.js and MySQL containers are attached to the network.

The application uses:

```text
DB_HOST=db
```

Docker's internal DNS resolves `db` to the database container.

This eliminates the need to manually configure container IP addresses.

## 12. Docker Hub Image Publishing

The application image was built with the tag:

```text
ivolve-app:lab8
```

Log in to Docker Hub:

```bash
docker login
```

Tag the image:

```bash
docker tag ivolve-app:lab8 nadafouuad/ivolve-app:lab8
```

Push:

```bash
docker push nadafouuad/ivolve-app:lab8
```

Docker Hub repository:

https://hub.docker.com/r/nadafouuad/ivolve-app

## 13. Key Takeaways

1. Docker Compose manages multiple interconnected containers using one YAML configuration.
2. Docker service names provide automatic DNS-based communication.
3. Custom bridge networks isolate application services from unrelated containers.
4. Named Docker volumes preserve application and database files across container recreation.
5. Health checks and `depends_on` improve service startup coordination.
6. Environment variables allow database configuration without hardcoding credentials in application source code.
7. Docker Hub provides a registry for distributing application images.
8. Docker Compose simplifies rebuilding, restarting, and managing multi-container applications.

## Conclusion

Successfully containerized a Node.js and MySQL application using Docker Compose.

The lab configured application-to-database connectivity, a dedicated Docker network, persistent MySQL data storage, and persistent application log storage.

The Docker Compose stack was started and recreated successfully while retaining its named volumes.

The application image was also prepared for Docker Hub distribution using the `nadafouuad/ivolve-app:lab8` tag.
