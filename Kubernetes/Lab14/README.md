# Lab 14: StatefulSet with Headless Service

## Objective

Deploy a MySQL database using a Kubernetes StatefulSet with persistent storage and a headless Service for stable DNS communication.

## What I Did

1. Used the existing `ivolve` namespace, ConfigMap (`ivolvecm`), and Secret (`ivolvesecret`).

2. Created a 1Gi Persistent Volume (`mysql-pv`) using local storage on `k8s-worker2`.

3. Created a headless Service named `ivolvedbsvc` with `clusterIP: None`.

4. Deployed a StatefulSet named `ivolvedb` with one MySQL replica.

5. Configured the Pod to use:
   - Environment variables from the ConfigMap and Secret
   - Node selector `project=ivolve`
   - Toleration `workload=ivolveApp:NoSchedule`
   - Persistent storage mounted at `/var/lib/mysql`

6. Used `volumeClaimTemplates` to automatically create a PVC for the MySQL Pod.

7. Connected to MySQL and verified that the `ivolvedb` database was created.

8. Created a `students` table and inserted sample records.

9. Deleted the MySQL Pod and verified that the StatefulSet recreated it and the database records were still available.

10. Tested DNS resolution using a temporary BusyBox Pod.

## Verification

Checked the StatefulSet and storage:

```bash
kubectl get statefulset -n ivolve
kubectl get pods -n ivolve -o wide
kubectl get pvc -n ivolve
kubectl get pv
```

Connected to MySQL:

```bash
kubectl exec -it ivolvedb-0 -n ivolve -- mysql -u root -p
```

Verified the database records:

```sql
USE ivolvedb;
SELECT * FROM students;
```

Tested DNS resolution:

```bash
kubectl exec dns-test -n ivolve -- nslookup ivolvedb-0.ivolvedbsvc.ivolve.svc.cluster.local
```

## What I Learned

- StatefulSets provide stable Pod names and persistent storage associations.
- Headless Services enable DNS discovery of individual StatefulSet Pods.
- ConfigMaps and Secrets provide configuration and credentials to containers.
- Node selectors control where Pods can be scheduled.
- Tolerations allow Pods to run on nodes with matching taints.
- VolumeClaimTemplates create dedicated PVCs for StatefulSet Pods.
- Persistent storage allows database records to survive Pod recreation.
- Local Persistent Volumes are tied to specific nodes and don't provide protection against node failure.

## Files

- `mysql-pv.yaml` — Persistent Volume configuration
- `mysql-statefulset.yaml` — MySQL StatefulSet configuration
- `mysql-headless-service.yaml` — Headless Service configuration
