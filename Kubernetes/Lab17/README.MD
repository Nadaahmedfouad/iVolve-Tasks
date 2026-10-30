# Lab 17: Control Pod-to-Pod Traffic via NetworkPolicy

## Objective

Configure a Kubernetes NetworkPolicy to restrict access to the MySQL Pod, allowing incoming connections only from the Node.js application Pods on TCP port 3306.

## What I Did

1. Checked the labels of the existing application and MySQL Pods.

2. Created a NetworkPolicy named `ivolvenp` in the `ivolve` namespace.

3. Configured the policy to:
   - Target Pods labeled `app=ivolvedb`
   - Allow ingress only from Pods labeled `app=ivolveapp`
   - Allow connections only on TCP port 3306
   - Leave egress traffic unrestricted by this policy

4. Applied the NetworkPolicy and verified its configuration.

5. Tested the Node.js application's connection to MySQL through the `/health` endpoint.

6. Created a temporary BusyBox Pod without the allowed application label and attempted to connect to MySQL.

## Verification

### NetworkPolicy

```bash
kubectl get networkpolicy -n ivolve
kubectl describe networkpolicy ivolvenp -n ivolve
```

### Authorized Traffic

```bash
curl -i http://10.104.40.251/health
```

Result: HTTP 200 OK.

The Node.js application successfully communicated with MySQL.

### Unauthorized Traffic

Created a temporary test Pod:

```bash
kubectl run unauthorized-test \
  -n ivolve \
  --image=busybox:1.36 \
  --restart=Never \
  --command -- sleep 3600
```

Attempted to connect to MySQL:

```bash
kubectl exec unauthorized-test -n ivolve -- \
  nc -z -w 3 ivolvedbsvc 3306
```

Result:

```text
command terminated with exit code 1
```

The connection failed, consistent with the NetworkPolicy blocking unauthorized traffic.

## What I Learned

- NetworkPolicies control network traffic between Kubernetes Pods.
- Pod selectors use labels to identify protected and allowed Pods.
- Ingress controls incoming traffic, while egress controls outgoing traffic.
- NetworkPolicies can restrict access to specific TCP or UDP ports.
- Calico enforces NetworkPolicies in our Kubernetes cluster.
- Kubernetes NetworkPolicies are additive, meaning other policies may allow additional traffic.

## Files

- `ivolve-np.yaml` — MySQL NetworkPolicy configuration
- `screenshots/` — Policy configuration and connectivity test results
