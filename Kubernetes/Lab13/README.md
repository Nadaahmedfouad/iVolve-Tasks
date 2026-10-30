# Lab 13: Persistent Storage Setup for Application Logging

## Objective

Set up persistent storage in Kubernetes to store application logs, ensuring that data is not lost when a Pod is deleted or recreated.

## What I Did

1. Created a Persistent Volume (PV) named `ivolvepv` with:
   - 1Gi storage capacity
   - hostPath at `/mnt/app-logs`
   - ReadWriteMany access mode
   - Retain reclaim policy
   - No StorageClass

2. Created a Persistent Volume Claim (PVC) named `ivolvepvc` requesting 1Gi of storage with ReadWriteMany access.

3. Verified that the PV and PVC were successfully bound.

4. Created a BusyBox Pod that mounted the PVC at `/app/logs`.

5. Wrote a sample log file inside the container and verified that it appeared on the worker node.

6. Deleted and recreated the Pod on the same worker node to test data persistence.

## Verification

Checked the PV and PVC:

```bash
kubectl get pv
kubectl get pvc
```

Both showed `Bound` status.

Created and read a sample log:

```bash
kubectl exec logging-test -- sh -c 'echo "Application started successfully" > /app/logs/app.log'
kubectl exec logging-test -- cat /app/logs/app.log
```

Verified the file on the worker node:

```bash
sudo cat /mnt/app-logs/app.log
```

The log remained available after recreating the Pod on the same worker.

## What I Learned

- PVs represent storage resources, while PVCs request storage.
- Kubernetes binds PVCs to compatible PVs.
- Persistent storage allows data to survive Pod deletion.
- The Retain policy preserves the underlying data when the PVC is deleted.
- hostPath uses a local directory on a Kubernetes node, so it is not suitable for shared storage across different nodes.
- Storage can be configured manually without using a StorageClass.

## Files

- `ivolve-pv.yaml` — Persistent Volume configuration
- `ivolve-pvc.yaml` — Persistent Volume Claim configuration
- `test-pod.yaml` — Pod used to test persistent storage
