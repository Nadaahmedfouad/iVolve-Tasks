# Lab 21 – Jenkins CI/CD Pipeline

## Overview

In this lab, I created a Jenkins CI/CD pipeline to automate building, scanning, and deploying a Java application to Kubernetes.

The pipeline uses a Jenkins Docker agent to run the stages, DockerHub to store the application image, and my Kubernetes cluster to run the application.

I also configured SCM polling so Jenkins can automatically start a new build when changes are pushed to GitHub.

## Tools Used

- Jenkins
- Git and GitHub
- Maven
- Docker and DockerHub
- Trivy
- Kubernetes
- kubectl

## Pipeline Workflow

The pipeline follows these steps:

1. Clone the DemoApp source code from GitHub.
2. Verify that the required project files exist.
3. Build the Java application using Maven.
4. Build a Docker image.
5. Scan the image using Trivy.
6. Push the image to DockerHub.
7. Remove the local Docker image.
8. Test Kubernetes authentication and permissions.
9. Clone the deployment configuration from my GitHub repository.
10. Update the Kubernetes manifest with the new image tag.
11. Deploy the application to Kubernetes and verify the rollout.

Each Docker image is tagged with its Jenkins build number, such as `build-15`.

## Automatic Triggering

I enabled **Poll SCM** in Jenkins using:

    H/5 * * * *

Jenkins checks the GitHub repositories approximately every five minutes.

When a new commit is detected, the pipeline starts automatically without using Build Now.

I tested this successfully with Build #15, which started with:

    Started by an SCM change

The build finished successfully.

## Kubernetes Deployment

The application is deployed in the `ivolve` namespace.

The deployment configuration is stored in `deployment.yaml` and includes:

- A Deployment running one DemoApp replica.
- A ClusterIP Service exposing port 8080.

The Jenkins pipeline updates the image tag in its workspace copy of the manifest before applying it to Kubernetes.

To verify the deployment:

    kubectl get deployment demoapp -n ivolve
    kubectl get pods -n ivolve -l app=demoapp
    kubectl get svc demoapp-service -n ivolve

To check the deployed image:

    kubectl get deployment demoapp -n ivolve -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'

## Challenges Faced

During testing, I faced two main issues:

**Kubernetes rollout timeout**

One build timed out while waiting for the new Pod to become ready. I checked the Pod status and kubelet logs to investigate the worker-node issues. Kubernetes eventually completed the deployment.

**Kubernetes API connection timeout**

Another build failed during the Kubernetes access check because of a TLS handshake timeout. I checked the cluster health and tested connectivity from both Windows and the Jenkins agent container.

After verifying connectivity, I tested the pipeline again successfully.

## Final Result

Build #15 completed successfully and was automatically triggered by an SCM change.

The pipeline can build the application, scan its Docker image, push it to DockerHub, and deploy it to Kubernetes.

## Repositories

**Application source code:**  
https://github.com/Ibrahim-Adel15/DemoApp

**Lab files and deployment configuration:**  
https://github.com/Nadaahmedfouad/iVolve-Tasks

**DockerHub image:**  
https://hub.docker.com/r/nadafouuad/demoapp

## Notes

- Jenkins runs locally using Docker Desktop, so the machine, Jenkins services, and Kubernetes cluster must be running for the pipeline to work.
- The current Trivy scan reports vulnerabilities but does not fail the build based on its findings.
- The Kubernetes image tag is updated in the Jenkins workspace, not committed back to GitHub.
- The temporary Kubernetes ServiceAccount token needs to be renewed or replaced with a suitable credential setup before using the pipeline long-term.
