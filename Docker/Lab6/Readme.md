# Lab 6: Docker Environment Variables and Resource Management

## Objective

Containerize a Python Flask application using Docker, configure environment variables through different methods, and manage container CPU and memory resources.

This lab demonstrates how to configure applications dynamically across staging, production, and development environments without modifying the application source code.

## Technologies Used

- Docker
- Python 3.12
- Flask
- Docker Environment Variables
- Docker Resource Limits
- PowerShell

## 1. Clone the Repository

Clone the application repository:

```bash
git clone https://github.com/Ibrahim-Adel15/PythonApp.git
cd PythonApp
```

The Flask application reads two environment variables:

- `APP_NAME`: Application name
- `APP_ENV`: Deployment environment

Both variables are required for the application to start.

The application exposes two endpoints:

| Endpoint  | Description                                               |
| --------- | --------------------------------------------------------- |
| `/`       | Returns application information and environment variables |
| `/health` | Returns application health status                         |

The application runs on port `5000`.

## 2. Create the Dockerfile

Create a file named `Dockerfile`:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

### Dockerfile Explanation

- `FROM`: Uses the lightweight Python 3.12 image.
- `WORKDIR`: Sets the application working directory.
- `COPY`: Copies application files into the image.
- `RUN`: Installs the required Python dependencies.
- `EXPOSE`: Documents the application's port.
- `CMD`: Starts the Flask application.

## 3. Build the Docker Image

Build the image and tag it as `ivolve1`:

```bash
docker build -t ivolve1 .
```

Verify the image:

```bash
docker images ivolve1
```

## 4. Run Container with Environment Variables

Create a container named `container1` using command-line environment variables.

```powershell
docker run -d `
  --name container1 `
  -p 5000:5000 `
  -e APP_NAME=iVolveApp1 `
  -e APP_ENV=stg `
  --cpus="1" `
  --memory="256m" `
  ivolve1
```

### Configuration

| Parameter        | Value      |
| ---------------- | ---------- |
| Container name   | container1 |
| Application name | iVolveApp1 |
| Environment      | stg        |
| Host port        | 5000       |
| Container port   | 5000       |
| CPU limit        | 1 CPU      |
| Memory limit     | 256 MiB    |

### Resource Management

The following flags restrict container resources:

```bash
--cpus="1"
--memory="256m"
```

`--cpus="1"` limits the container to one CPU's worth of processing capacity.

`--memory="256m"` restricts the container's memory allocation to approximately 256 MiB.

These limits help prevent a single container from consuming excessive host resources.

## 5. Verify the Application

Access the application:

```text
http://localhost:5000
```

The application returned:

```json
{
  "application": "iVolveApp1",
  "environment": "stg",
  "message": "Welcome to the Docker Env & Resource Management Lab!"
}
```

Check the health endpoint:

```text
http://localhost:5000/health
```

Expected response:

```json
{
  "status": "healthy"
}
```

## 6. Verify Environment Variables

Check the environment variables inside `container1`:

```bash
docker exec container1 printenv APP_NAME APP_ENV
```

Output:

```text
iVolveApp1
stg
```

This confirms that Docker successfully injected the variables into the container.

## 7. Monitor Container Resources

Monitor CPU and memory usage:

```bash
docker stats container1
```

Alternatively, display a single snapshot:

```bash
docker stats container1 --no-stream
```

The command displays:

- CPU usage percentage
- Memory usage and limit
- Memory utilization percentage
- Network input/output
- Block input/output
- Number of running processes

The configured memory limit is approximately **256 MiB**.

Resource limits can also be inspected using:

```bash
docker inspect container1 --format '{{.HostConfig.NanoCpus}} {{.HostConfig.Memory}}'
```

Expected configured values:

```text
1000000000 268435456
```

These represent one CPU and 256 MiB of memory.

## 8. Configure Environment Variables Using a File

Create an `env.txt` file containing:

```env
APP_NAME=iVolveApp2
APP_ENV=prod
```

This approach separates environment-specific configuration from the Docker command.

Stop and remove the previous container to free port 5000:

```bash
docker stop container1
docker rm container1
```

Run a new container using the environment file:

```bash
docker run -d --name container2 -p 5000:5000 --env-file env.txt ivolve1
```

### Verify Environment Variables

```bash
docker exec container2 printenv APP_NAME APP_ENV
```

Output:

```text
iVolveApp2
prod
```

Access:

```text
http://localhost:5000
```

Application response:

```json
{
  "application": "iVolveApp2",
  "environment": "prod",
  "message": "Welcome to the Docker Env & Resource Management Lab!"
}
```

The application successfully loaded the production environment configuration from `env.txt`.

## 9. Configure Environment Variables Inside the Dockerfile

Create a new file named `Dockerfile.dev`.

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

ENV APP_NAME=iVolveApp3
ENV APP_ENV=dev

EXPOSE 5000

CMD ["python", "app.py"]
```

### Explanation

The `ENV` instruction defines default environment variables directly in the Docker image.

These values are automatically available to containers created from the image unless overridden at runtime.

## 10. Build the Development Image

Build:

```bash
docker build -f Dockerfile.dev -t ivolve3 .
```

Verify:

```bash
docker images ivolve3
```

**Result:** The image was successfully built with a size of approximately **208 MB**.

## 11. Run the Development Container

Remove the previous `container2`:

```bash
docker rm -f container2
```

Run the development image:

```bash
docker run -d --name container2 -p 5000:5000 ivolve3
```

Notice that no environment variables are supplied using `-e` or `--env-file`.

The values are already defined inside `Dockerfile.dev`.

### Verify the Variables

```bash
docker exec container2 printenv APP_NAME APP_ENV
```

Output:

```text
iVolveApp3
dev
```

Access:

```text
http://localhost:5000
```

Application response:

```json
{
  "application": "iVolveApp3",
  "environment": "dev",
  "message": "Welcome to the Docker Env & Resource Management Lab!"
}
```

The application successfully started using the environment variables defined in the Dockerfile.

## 12. Environment Variable Methods Comparison

| Method            | APP_NAME   | APP_ENV | Configuration Source |
| ----------------- | ---------- | ------- | -------------------- |
| Command-line `-e` | iVolveApp1 | stg     | Docker run arguments |
| `--env-file`      | iVolveApp2 | prod    | External env.txt     |
| Dockerfile `ENV`  | iVolveApp3 | dev     | Image configuration  |

### Key Differences

**Command-line environment variables (`-e`)**

Useful for providing configuration dynamically when starting individual containers.

**Environment files (`--env-file`)**

Useful for managing multiple environment variables and keeping configuration separate from Docker commands.

**Dockerfile environment variables (`ENV`)**

Useful for defining default, non-sensitive application configuration within the Docker image.

Runtime environment variables can override defaults defined using `ENV`.

Sensitive information such as passwords, API keys, and access tokens should not be hardcoded inside Dockerfiles.

```

## 12. Key Takeaways

1. Docker environment variables allow application configuration without source-code modifications.
2. The `-e` flag passes individual variables to containers.
3. The `--env-file` flag loads multiple variables from external files.
4. The Dockerfile `ENV` instruction defines default values inside an image.
5. `--cpus` and `--memory` restrict container resource consumption.
6. `docker stats` provides real-time CPU and memory monitoring.
7. `docker exec` allows verification of environment variables inside running containers.
8. Environment-based configuration supports deployment across development, staging, and production environments.

```
