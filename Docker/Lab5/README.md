# Lab 5: Docker Image Vulnerability Scanning with Trivy

## Objective

Scan a Docker image for security vulnerabilities using Trivy, identify HIGH and CRITICAL vulnerabilities, generate HTML security reports, and remediate the identified issues by updating the base image and application dependencies.

## Technologies Used

- Docker
- Trivy v0.76.0
- Node.js 18 and 24
- Express.js
- Alpine Linux
- PowerShell

## 1. Clone the Repository

```bash
git clone https://github.com/Ibrahim-Adel15/TrivyImage.git
cd TrivyImage
```

## 2. Build the Original Docker Image

The original Dockerfile:

```dockerfile
FROM node:18-bullseye

WORKDIR /app

COPY package.json ./

RUN npm install

COPY app.js ./

EXPOSE 3000

CMD ["npm", "start"]
```

Build the image:

```bash
docker build -t trivy-lab:v1 .
```

Verify:

```bash
docker images trivy-lab
```

## 3. Install and Verify Trivy

Trivy was installed on Windows using Windows Package Manager.

```powershell
winget install --id AquaSecurity.Trivy -e
```

Verify installation:

```bash
trivy --version
```

**Installed version:** 0.76.0

## 4. Scan the Original Docker Image

Scan for all vulnerabilities:

```bash
trivy image --scanners vuln trivy-lab:v1
```

Scan only HIGH and CRITICAL vulnerabilities:

```bash
trivy image --scanners vuln --severity HIGH,CRITICAL trivy-lab:v1
```

Export the scan results to JSON:

```bash
trivy image --scanners vuln --severity HIGH,CRITICAL --format json -o scan-v1.json trivy-lab:v1
```

### Initial Security Findings

The original image used Node.js 18 with Debian 11 (Bullseye) and Express 4.18.2.

The scan reported:

- An outdated operating system containing numerous known vulnerabilities.
- 26 HIGH vulnerabilities in Node.js packages.
- 1 CRITICAL vulnerability in Node.js packages.
- Multiple outdated application and bundled npm dependencies.
- A warning that Debian 11 was no longer supported by the distribution.

Some identified vulnerable packages included:

| Package         | Vulnerability                                 |
| --------------- | --------------------------------------------- |
| body-parser     | Denial of Service                             |
| brace-expansion | Denial of Service                             |
| cross-spawn     | Regular Expression Denial of Service          |
| path-to-regexp  | Regular Expression Denial of Service          |
| tar             | Denial of Service and unsafe archive handling |

These findings demonstrated the need to upgrade both the Docker base image and application dependencies.

## 5. Generate the Initial HTML Report

Download the Trivy HTML template:

```powershell
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl" -OutFile "html.tpl"
```

Generate the report:

```bash
trivy image --scanners vuln --severity HIGH,CRITICAL --format template --template "@html.tpl" -o report-v1.html trivy-lab:v1
```

Open the report:

```powershell
start report-v1.html
```

## 6. Vulnerability Remediation

Several improvements were implemented to reduce the application's attack surface and eliminate detected vulnerabilities.

### 6.1 Upgrade Application Dependencies

The original application used Express 4.18.2.

Update Express within the application directory:

```bash
npm install express@4 --save
```

Verify:

```bash
npm list express
```

**Updated version:** Express 4.22.3

The updated `package-lock.json` was retained to ensure reproducible dependency installation.

### 6.2 Upgrade the Base Image

The outdated base image was replaced:

```text
Before: node:18-bullseye
After:  node:24-alpine
```

This moved the application to Node.js 24 with Alpine Linux, providing a newer runtime and a smaller operating system footprint.

### 6.3 Implement Multi-Stage Builds

A new file named `Dockerfile.v2` was created.

```dockerfile
# Stage 1: Install dependencies
FROM node:24-alpine AS builder

WORKDIR /app

COPY package*.json ./

RUN npm ci --omit=dev

# Stage 2: Runtime
FROM node:24-alpine

WORKDIR /app

ENV NODE_ENV=production

# Remove unnecessary package managers
RUN rm -rf /usr/local/lib/node_modules/npm \
    /usr/local/lib/node_modules/corepack \
    /opt/yarn-v1.22.22 \
    /usr/local/bin/npm \
    /usr/local/bin/npx \
    /usr/local/bin/corepack \
    /usr/local/bin/yarn \
    /usr/local/bin/yarnpkg

COPY --from=builder --chown=node:node /app/node_modules ./node_modules

COPY --chown=node:node package.json app.js ./

USER node

EXPOSE 3000

CMD ["node", "app.js"]
```

### Security Improvements

**Multi-stage build:** Separates dependency installation from the final runtime environment.

**Production-only dependencies:** Uses `npm ci --omit=dev` to install only production dependencies.

**Reduced attack surface:** Removes unused npm, Yarn, and Corepack tooling from the runtime filesystem.

**Non-root execution:** Uses the built-in `node` user instead of running the application as root.

**Dependency updates:** Upgrades Express and its transitive dependencies to versions without the originally detected application dependency vulnerabilities.

## 7. Build the Remediated Image

Build the updated image:

```bash
docker build --pull --no-cache -f Dockerfile.v2 -t ivolve-app:v2 .
```

Verify:

```bash
docker images ivolve-app
```

## 8. Rescan the Updated Image

Run the HIGH and CRITICAL vulnerability scan:

```bash
trivy image --scanners vuln --severity HIGH,CRITICAL ivolve-app:v2
```

The initial v2 image scan showed four HIGH vulnerabilities related to bundled npm dependencies.

After removing the unused package managers from the runtime image and rebuilding, the final Trivy scan reported:

- Alpine OS: 0 HIGH/CRITICAL vulnerabilities.
- Node.js packages: 0 HIGH/CRITICAL vulnerabilities.

### Vulnerability Comparison

| Metric                            | Original v1      | Final v2       |
| --------------------------------- | ---------------- | -------------- |
| Base image                        | node:18-bullseye | node:24-alpine |
| Operating system                  | Debian 11        | Alpine 3.24.2  |
| Express version                   | 4.18.2           | 4.22.3         |
| Node.js package HIGH findings     | 26               | 0              |
| Node.js package CRITICAL findings | 1                | 0              |
| OS HIGH/CRITICAL findings         | Present          | 0              |
| Runtime user                      | Root             | Non-root       |
| Build method                      | Single-stage     | Multi-stage    |

**Result: 100% reduction in detected HIGH and CRITICAL vulnerabilities under the selected scan filters.**

## 9. Generate the Final HTML Report

```bash
trivy image --scanners vuln --severity HIGH,CRITICAL --format template --template "@html.tpl" -o report-v2.html ivolve-app:v2
```

Open the final report:

```powershell
start report-v2.html
```

The final HTML report confirmed that no HIGH or CRITICAL vulnerabilities were detected in the scanned Alpine packages or Node.js dependencies.

Save the scan results:

```bash
trivy image --scanners vuln --severity HIGH,CRITICAL --format json -o scan-v2.json ivolve-app:v2
```

## 10. Application Verification

Run the updated container:

```bash
docker run -d --name trivy-v2 -p 3000:3000 ivolve-app:v2
```

Check application logs:

```bash
docker logs trivy-v2
```

Verify the runtime user:

```bash
docker exec trivy-v2 id
```

The expected UID must be nonzero, confirming that the container runs without root privileges.

Access the application:

```text
http://localhost:3000
```

Remove the test container:

```bash
docker rm -f trivy-v2
```

## 11. Key Takeaways

1. Trivy identifies known security vulnerabilities in container operating systems and application dependencies.
2. Outdated base images introduce significant security risks.
3. Updating application dependencies helps eliminate known vulnerable packages.
4. Multi-stage Docker builds reduce unnecessary runtime components.
5. Removing unused tools minimizes the runtime attack surface.
6. Running containers as non-root users improves security.
7. Rescanning after remediation verifies whether security improvements were effective.
8. Vulnerability scanning should be integrated into CI/CD pipelines to continuously assess container security.
