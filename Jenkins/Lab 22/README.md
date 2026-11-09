# Lab 22: Jenkins CI/CD Pipeline with Shared Libraries

## Overview

In this lab, I updated the Jenkins pipeline from Lab 21 to use Shared Libraries. Instead of keeping all the build and deployment commands inside the Jenkinsfile, I moved them into reusable Groovy functions stored in a separate GitHub repository.

The pipeline runs on a custom Jenkins agent and deploys the application to Kubernetes.

## Tools Used

- Jenkins and Jenkins Agent
- Git and GitHub
- Maven
- Docker and Docker Hub
- Trivy
- Kubernetes
- Groovy

## Shared Libraries

I created a separate GitHub repository for the Shared Library.

The repository contains three functions:

- **buildApp.groovy:** Builds the Java application and runs Maven tests.
- **buildAndPushImage.groovy:** Builds the Docker image, scans it using Trivy, pushes it to Docker Hub, and removes the local image.
- **deployToK8s.groovy:** Updates the Kubernetes deployment file, deploys the application, and checks the rollout.

I registered the repository in Jenkins under Global Pipeline Libraries using the name `shared-lib`.

## Pipeline Workflow

The Jenkinsfile loads the Shared Library using:

`@Library('shared-lib') _`

The pipeline then:

1. Clones the DemoApp repository.
2. Verifies the application files.
3. Builds the Java application using `buildApp()`.
4. Builds, scans, and pushes the Docker image using `buildAndPushImage()`.
5. Retrieves the Kubernetes deployment file.
6. Deploys the application using `deployToK8s()`.

All stages run using the existing `ivolve-agent`.

## Result

The Jenkins pipeline completed successfully.

The application was built, the Docker image was scanned and pushed to Docker Hub, and the updated application was deployed successfully to Kubernetes.

## Screenshots

Screenshots are included to show the successful pipeline, Jenkins agent, Shared Library configuration, and execution results.
