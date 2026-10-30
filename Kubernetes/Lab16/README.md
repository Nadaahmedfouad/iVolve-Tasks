# Lab 16: Pod Resource Management with CPU and Memory Requests and Limits

## Objective

Configure CPU and memory requests and limits for the existing Node.js Deployment and monitor actual resource usage using Kubernetes Metrics Server.

## What I Did

1. Checked the available CPU and memory on `k8s-worker2`.

2. Increased the VirtualBox VM resources to 4 vCPUs and 6 GB RAM to support the new resource requirements.

3. Updated the existing `ivolveapp` Deployment with the following resources for each Node.js container:

   | Resource | Request | Limit  |
   | -------- | ------- | ------ |
   | CPU      | 1 CPU   | 2 CPUs |
   | Memory   | 1Gi     | 2Gi    |

4. Applied the updated Deployment configuration.

5. Encountered a rollout issue because the existing `ivolvequota` ResourceQuota allowed only 3 Pods.

6. Increased the Pod quota to 4 and triggered a new rollout.

7. Verified the resource requests and limits using `kubectl describe`.

8. Installed Metrics Server to monitor CPU and memory usage.

9. Fixed a Kubelet certificate verification issue in the local lab environment.

10. Used `kubectl top pods` to check actual resource consumption.

## Verification

### Resource Requests and Limits

```bash
kubectl describe pods -n ivolve -l app=ivolveapp
```

Verified the following configuration:

```yaml
resources:
  requests:
    cpu: "1"
    memory: "1Gi"
  limits:
    cpu: "2"
    memory: "2Gi"
```

### Deployment Status

```bash
kubectl get deployment ivolveapp -n ivolve
```

Verified that the Deployment successfully rolled out with the updated resource configuration.

### Resource Monitoring

```bash
kubectl top pods -n ivolve
```

Metrics Server successfully returned CPU and memory usage for the application and MySQL Pods.

## Issues Faced

**1. Deployment rollout failure**

The existing ResourceQuota allowed only 3 Pods, preventing Kubernetes from creating an additional Pod during the rolling update.

Updated the quota:

```bash
kubectl patch resourcequota ivolvequota -n ivolve \
  --type merge -p '{"spec":{"hard":{"pods":"4"}}}'
```

Triggered a new rollout:

```bash
kubectl rollout restart deployment/ivolveapp -n ivolve
```

**2. Metrics Server not ready**

Metrics Server couldn't collect node metrics because the Kubelet certificates didn't contain the required IP Subject Alternative Names.

For this local training cluster, added `--kubelet-insecure-tls` to the Metrics Server arguments.

This is a lab-only workaround and is not recommended for production.

## What I Learned

- Requests specify the resources Kubernetes uses for scheduling.
- Limits control the maximum CPU and memory a container can use.
- CPU usage can be throttled when a container reaches its CPU limit.
- Containers may be OOMKilled if they exceed their memory limits.
- ResourceQuotas can prevent Deployments from creating additional Pods.
- Metrics Server provides CPU and memory usage through `kubectl top`.
- Actual resource usage can be much lower than configured requests.

## Files

- `ivolve-deployment.yaml` — Updated Deployment configuration with CPU and memory requests and limits.
- `screenshots/` — Deployment status, configured resources, and actual resource usage.
