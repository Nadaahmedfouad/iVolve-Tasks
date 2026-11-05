# Lab 23: GitOps Workflow with Jenkins and Argo CD

## Objective

Implement a GitOps workflow where Jenkins handles Continuous Integration (CI) and Argo CD handles Continuous Deployment (CD) to Kubernetes.

## Tools Used

- Jenkins
- Jenkins Shared Library
- Docker and Docker Hub
- Trivy
- GitHub
- Argo CD
- Kubernetes

## Implementation

### 1. Argo CD Installation

Installed Argo CD in the Kubernetes cluster under the `argocd` namespace and accessed its web dashboard.

### 2. GitOps Repository

Configured Argo CD to monitor the GitHub repository `Nadaahmedfouad/iVolve-Tasks`, specifically the `Jenkins/Lab23` directory.

Created an Argo CD Application named `demoapp` with automatic synchronization enabled.

### 3. Jenkins CI Pipeline

Reused the existing shared-library functions to:

- Clone the DemoApp repository.
- Build the application using Maven.
- Build the Docker image.
- Scan the image using Trivy.
- Push the image to Docker Hub.

### 4. GitOps Shared Library

Created a new shared-library function called `updateGitOpsRepo.groovy`.

Instead of deploying directly using `kubectl apply`, Jenkins now:

- Updates the Docker image tag in `deployment.yaml`.
- Commits the updated file.
- Pushes the changes to GitHub.

Argo CD detects the Git changes and automatically synchronizes the deployment with Kubernetes.

### 5. Deployment Verification

Triggered the Jenkins pipeline successfully and verified that Argo CD synchronized the updated deployment.

**Final Results:**

- Jenkins Build #1: SUCCESS
- Argo CD Sync Status: Synced
- Argo CD Health Status: Healthy
- Deployed Image: `nadafouuad/demoapp:build-1`
- Kubernetes Node: `k8s-worker2`

## Screenshots

Screenshots are available in the `Screenshots` folder, showing the Jenkins pipeline, Argo CD application, and deployment verification.

## Conclusion

Successfully implemented a GitOps workflow using Jenkins for CI and Argo CD for CD. Deployment changes are now managed through GitHub instead of being applied directly by Jenkins.
