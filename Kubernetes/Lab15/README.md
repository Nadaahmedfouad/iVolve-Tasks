# Lab 15: Node.js Application Deployment with ClusterIP Service

## Objective

Deploy a Node.js application on Kubernetes with two replicas, connect it to the MySQL database from Lab 14, and expose it internally using a ClusterIP Service.

## What I Did

1. Used my custom Docker image `nadafouuad/ivolve-app:lab8` from Docker Lab 8.

2. Updated the existing ConfigMap to use `ivolvedbsvc` as the MySQL hostname.

3. Created a Persistent Volume and PVC for storing application logs.

4. Created a Deployment named `ivolveapp` in the `ivolve` namespace with two replicas.

5. Configured the Pods with:
   - Environment variables from `ivolvecm` and `ivolvesecret`
   - Node selector `project=ivolve`
   - Toleration `workload=ivolveApp:NoSchedule`
   - Persistent storage mounted at `/app/logs`

6. Added a MySQL init container that waits for MySQL, creates the `ivolve` database if needed, and grants permissions to the application user.

7. Added a readiness probe using the application's `/ready` endpoint.

8. Created a ClusterIP Service named `ivolveappsvc` to distribute traffic between the two application replicas.

9. Verified the database connection, MySQL user permissions, application health endpoint, and persistent access logs.

## Verification

### Deployment and Service

```bash
kubectl get deployment ivolveapp -n ivolve
kubectl get pods -n ivolve -o wide
kubectl get svc ivolveappsvc -n ivolve
```

Both application replicas were running and ready.

The ClusterIP Service was created on port 80 and forwarded traffic to the Node.js containers on port 3000.

### Application Health

```bash
curl -i http://10.104.40.251/health
```

The application returned HTTP 200, confirming successful communication with MySQL.

### Database Verification

Connected to MySQL:

```bash
kubectl exec -it ivolvedb-0 -n ivolve -- mysql -u root -p
```

Checked the database and permissions:

```sql
SHOW DATABASES;
SHOW GRANTS FOR 'ivolve_user'@'%';
```

Confirmed that the `ivolve` database exists and the application user has the required permissions.

### Persistent Logs

Checked the log file on `k8s-worker2`:

```bash
sudo tail -n 10 /mnt/ivolve-app-logs/access.log
```

The log contained successful HTTP requests from Kubernetes readiness probes.

## What I Learned

- Deployments manage multiple replicas of an application.
- Init containers perform setup tasks before the main application starts.
- ConfigMaps and Secrets provide application configuration and credentials.
- Node selectors and tolerations control Pod scheduling.
- Persistent Volumes allow application logs to be stored outside the container filesystem.
- Readiness probes determine whether a Pod is ready to receive traffic.
- ClusterIP Services provide internal access and distribute traffic between application Pods.
- Kubernetes applications can communicate with databases using Service DNS names.

## Files

- `app-pv.yaml` — Application Persistent Volume
- `app-pvc.yaml` — Application Persistent Volume Claim
- `ivolve-deployment.yaml` — Node.js Deployment and MySQL init container
- `ivolve-app-service.yaml` — ClusterIP Service
