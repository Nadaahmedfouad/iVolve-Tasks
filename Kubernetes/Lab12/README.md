# Lab 12: Managing Configuration and Sensitive Data with ConfigMaps and Secrets

## Objective

The goal of this lab is to create a ConfigMap and a Secret in Kubernetes to store MySQL configuration and credentials separately.

## Steps

### 1. Create the ConfigMap

I created a ConfigMap named `ivolvecm` inside the `ivolve` namespace to store the database host and username.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: ivolvecm
  namespace: ivolve
data:
  DB_HOST: "mysql"
  DB_USER: "ivolve_user"
```

Applied it using:

```bash
kubectl apply -f ivolvecm.yaml
```

### 2. Encode the Passwords

Before creating the Secret, I encoded two test passwords using Base64.

```bash
echo -n 'RootPass123!' | base64
echo -n 'AppPass123!' | base64
```

Base64 is encoding, not encryption. It is used here because Kubernetes expects Base64-encoded values under the Secret's `data` field.

### 3. Create the Secret

I created a Secret named `ivolvesecret` in the same namespace to store the MySQL root password and application database password.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: ivolvesecret
  namespace: ivolve
type: Opaque
data:
  MYSQL_ROOT_PASSWORD: <base64-encoded-root-password>
  DB_PASSWORD: <base64-encoded-app-password>
```

Applied it using:

```bash
kubectl apply -f ivolvesecret.yaml
```

The passwords used were only for testing.

### 4. Verify the Resources

I checked that both resources were created successfully:

```bash
kubectl get configmaps,secrets -n ivolve
```

Then checked their details:

```bash
kubectl describe configmap ivolvecm -n ivolve
kubectl describe secret ivolvesecret -n ivolve
```

The ConfigMap displayed the database configuration, while the Secret showed the stored keys and their sizes without displaying the passwords.

## Result

Successfully created and verified a ConfigMap and a Secret inside the `ivolve` namespace.

The MySQL configuration and credentials are now stored separately and can be referenced by applications running in Kubernetes.

## What I Learned

- ConfigMaps store non-sensitive application configuration.
- Secrets are designed to store sensitive data such as passwords.
- Kubernetes Secrets can use Base64-encoded values in YAML manifests.
- Base64 encoding is not encryption.
- Both ConfigMaps and Secrets can be used by Pods through environment variables or mounted files.
