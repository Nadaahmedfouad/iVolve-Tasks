# Lab 3: Creating a Docker Image for a Java Application

## Objective

Containerize a Java calculator application using Docker and compare two approaches:

1. **Build Inside Docker:** Build the application using Maven inside a Docker image.
2. **Build Locally:** Build the application locally using Maven and copy the generated JAR into a lightweight Java runtime image.

The objective is to understand Docker image creation, container execution, non-root users, and image size optimization.

## Tools & Technologies

- Docker Desktop
- Java JDK 17
- Apache Maven 3.9.11
- Docker CLI
- Alpine Linux
- Windows PowerShell

**Source Repository:** [CalculatorMavenApp](https://github.com/Ibrahim-Adel15/CalculatorMavenApp)

## Part 1: Building Inside Docker

### 1. Dockerfile.app1

```dockerfile
FROM maven:3.9-eclipse-temurin-17-alpine

WORKDIR /app

COPY pom.xml .
COPY src ./src

RUN mvn clean package

RUN addgroup -S appgroup && \
    adduser -S -h /home/appuser -G appgroup appuser && \
    mkdir -p /home/appuser/.m2 && \
    chown -R appuser:appgroup /app /home/appuser

ENV HOME=/home/appuser
ENV MAVEN_CONFIG=/home/appuser/.m2

USER appuser

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "target/calculator.jar"]
```

### 2. Build the Docker Image

```bash
docker build -f Dockerfile.app1 -t app1 .
```

### 3. Run the Container

```bash
docker run -it --name container1 -p 8080:8080 app1
```

The `-it` flags enable interactive input because the calculator requires user input through the terminal.

### 4. Verify the Application

The calculator was tested using:

```text
Enter first number: 10
Enter second number: 20

Sum: 30.0
Subtract: -10.0
Multiply: 200.0
Divide: 0.5
```

### 5. Image Size

```bash
docker images app1
```

**Result: 588 MB**

### 6. Cleanup

```bash
docker rm container1
docker rmi app1
```

---

## Part 2: Building Locally and Packaging with Docker

### 1. Build Locally Using Maven

```bash
mvn clean package
```

The generated artifact is:

```text
target/calculator.jar
```

### 2. Dockerfile.app2

```dockerfile
FROM eclipse-temurin:17-jre-alpine

WORKDIR /app

RUN addgroup -S appgroup && \
    adduser -S -h /home/appuser -G appgroup appuser

COPY --chown=appuser:appgroup target/calculator.jar /app/calculator.jar

USER appuser

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "/app/calculator.jar"]
```

### 3. Build the Docker Image

```bash
docker build -f Dockerfile.app2 -t app2 .
```

### 4. Run the Container

```bash
docker run -it --name container2 -p 8080:8080 app2
```

### 5. Verify Application

The application executes inside the container using the Java 17 runtime.

### 6. Image Size

```bash
docker images app2
```

**Result: 265 MB**

### 7. Cleanup

```bash
docker rm container2
docker rmi app2
```

---

## Bonus: Docker Image Size Comparison

| Feature              | App1                  | App2                |
| -------------------- | --------------------- | ------------------- |
| Build approach       | Inside Docker         | Locally using Maven |
| Base image           | Maven + JDK 17 Alpine | Java 17 JRE Alpine  |
| Maven included       | Yes                   | No                  |
| JDK included         | Yes                   | No, JRE only        |
| Source code included | Yes                   | No                  |
| Image size           | 588 MB                | 265 MB              |

### Image Size Reduction

```text
588 MB - 265 MB = 323 MB

Reduction = (323 / 588) × 100 = 54.9%
```

**Result: App2 is approximately 54.9% smaller than App1.**

### Why Is App2 Smaller?

App1 contains Maven, the JDK, application source code, and compiled artifacts.

App2 contains only the Java runtime and the packaged application.

By excluding build-time tools and unnecessary development files from the final runtime image, App2 reduces storage requirements and image transfer overhead.

## Key Concepts Learned

- **Dockerfile:** Defines instructions for building a Docker image.
- **Base Image:** Provides the initial runtime environment.
- **Non-root User:** Improves container security by avoiding unnecessary root privileges.
- **Build Context:** Files made available to Docker during image creation.
- **ENTRYPOINT:** Defines the command executed when a container starts.
- **Interactive Mode (`-it`):** Enables terminal input for interactive applications.
- **JDK vs. JRE:** The JDK includes development tools, while the JRE provides the runtime environment.
- **Image Optimization:** Minimizes unnecessary dependencies to reduce image size.

**Note:** The Java calculator is a CLI application. Although port 8080 is declared for the lab, the application does not implement an HTTP service.

## Conclusion

Both Docker approaches successfully packaged the Java calculator application.

Building locally and copying the JAR into a JRE-based image reduced the image size from **588 MB to 265 MB**, demonstrating the benefits of separating build-time dependencies from runtime requirements.

**Lab 3 completed successfully.**
