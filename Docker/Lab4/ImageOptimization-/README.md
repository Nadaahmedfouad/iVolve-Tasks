# Lab 4: Docker Image Optimization and Export/Load

## Objective

Optimize a Docker image for a Java Spring Boot application using multi-stage builds, improve container security by running as a non-root user, and practice exporting and importing Docker images.

## Technologies Used

- Docker
- Java 17
- Maven
- Spring Boot 3.2.0
- Alpine Linux
- PowerShell

## 1. Clone the Repository

```bash
git clone https://github.com/Ibrahim-Adel15/ImageOptimization-.git
cd ImageOptimization-
```

## 2. Build the Original Docker Image

The original Dockerfile uses a Maven image to build and run the Spring Boot application.

```dockerfile
FROM maven:3.9.15-eclipse-temurin-17-alpine

WORKDIR /app
COPY . .
RUN mvn package

EXPOSE 8080

CMD ["java", "-jar", "target/demo-0.0.1-SNAPSHOT.jar"]
```

Build the image:

```bash
docker build -t image1 .
```

Check its size:

```bash
docker images image1
```

**Result: 671 MB**

Run the container:

```bash
docker run -d --name container1 -p 8080:8080 image1
```

Verify the application:

```bash
docker logs container1
```

The Spring Boot application started successfully on port 8080.

Stop and remove the container:

```bash
docker stop container1
docker rm container1
```

## 3. Optimize Using a Multi-Stage Dockerfile

Create a file named `Dockerfile.optimized`.

```dockerfile
# Stage 1: Build
FROM maven:3.9.15-eclipse-temurin-17-alpine AS builder

WORKDIR /app

COPY pom.xml .
RUN mvn dependency:go-offline -B

COPY src ./src
RUN mvn package -DskipTests

# Stage 2: Runtime
FROM eclipse-temurin:17-jre-alpine

WORKDIR /app

RUN addgroup -S appgroup && adduser -S appuser -G appgroup

COPY --from=builder --chown=appuser:appgroup /app/target/demo-0.0.1-SNAPSHOT.jar app.jar

USER appuser

EXPOSE 8080

CMD ["java", "-jar", "app.jar"]
```

### Optimization Techniques

**Multi-stage build:** Separates the build environment from the runtime environment.

**Smaller runtime image:** Uses Java JRE instead of including Maven and the full JDK.

**Layer caching:** Copies `pom.xml` before application source code to reuse dependency layers when possible.

**Non-root execution:** Creates a dedicated application user instead of running the container as root.

**Minimal runtime artifacts:** Copies only the packaged application JAR into the final image.

## 4. Build and Test the Optimized Image

Build:

```bash
docker build -f Dockerfile.optimized -t image2 .
```

Check the image size:

```bash
docker images image2
```

**Result: 302 MB**

Run the optimized container:

```bash
docker run -d --name container2 -p 8080:8080 image2
```

Verify application startup:

```bash
docker logs container2
```

Verify that the application runs as a non-root user:

```bash
docker exec container2 id
```

The UID should be different from `0`, confirming non-root execution.

## 5. Compare Image Sizes

| Metric              | Original Image | Optimized Image |
| ------------------- | -------------- | --------------- |
| Image name          | image1         | image2          |
| Image size          | 671 MB         | 302 MB          |
| Build approach      | Single-stage   | Multi-stage     |
| Runtime environment | Maven + JDK    | JRE             |
| Runtime user        | Root           | Non-root        |
| Application port    | 8080           | 8080            |

### Results

- **Original image:** 671 MB
- **Optimized image:** 302 MB
- **Size reduction:** 369 MB
- **Percentage reduction:** Approximately 55%

The optimized image is significantly smaller while preserving the application's Java 17 runtime requirements.

## 6. Export the Docker Image

Save the optimized image as a TAR archive:

```bash
docker save -o image2.tar image2:latest
```

This archive contains the image layers and metadata needed to restore the image.

## 7. Delete the Docker Image

Remove the container before deleting the image:

```bash
docker rm -f container2
```

Remove the optimized image:

```bash
docker rmi image2:latest
```

Verify deletion:

```bash
docker images image2
```

## 8. Import the Docker Image

Restore the saved image:

```bash
docker load -i image2.tar
```

Verify restoration:

```bash
docker images image2
```

The restored image retains its original name, tag, and image contents.

Test the restored image:

```bash
docker run -d --name container2-restored -p 8080:8080 image2
docker logs container2-restored
```

Remove the test container:

```bash
docker rm -f container2-restored
```

## 9. Key Takeaways

1. Multi-stage builds eliminate unnecessary build tools from the final runtime image.
2. Using a smaller runtime base image significantly reduces image size.
3. Running containers as non-root users improves security.
4. Docker layer caching helps avoid unnecessary dependency downloads during subsequent builds.
5. `docker save` and `docker load` allow images to be transferred and restored without rebuilding.
6. Optimized images reduce storage requirements and can improve deployment and distribution efficiency.
