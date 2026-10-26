# Lab 11: Namespace Management and ResourceQuota

## Objective

The goal of this lab is to create a Kubernetes namespace and set a ResourceQuota to limit the number of Pods that can run inside it.

I also tested the quota by creating three Pods and trying to create a fourth one.

## Steps

### 1. Create a Namespace

First, I created a namespace called `ivolve`:

```bash
kubectl create namespace ivolve
```

Then checked that it was created:

```bash
kubectl get namespaces
```

### 2. Create a ResourceQuota

I created a ResourceQuota named `ivolvequota` to limit the namespace to a maximum of 3 Pods.

```bash
kubectl create quota ivolvequota --hard=pods=3 -n ivolve
```

To verify the quota:

```bash
kubectl describe resourcequota ivolvequota -n ivolve
```

Initially, the quota showed 0 Pods used out of 3 allowed.

### 3. Create Three Pods

I created three Nginx Pods inside the namespace:

```bash
kubectl run pod1 --image=nginx -n ivolve
kubectl run pod2 --image=nginx -n ivolve
kubectl run pod3 --image=nginx -n ivolve
```

Then checked their status:

```bash
kubectl get pods -n ivolve -o wide
```

All three Pods were created successfully.

### 4. Test the ResourceQuota

After creating the three Pods, I checked the quota again:

```bash
kubectl describe resourcequota ivolvequota -n ivolve
```

The quota showed:

```text
Resource   Used   Hard
pods       3      3
```

I then tried to create a fourth Pod:

```bash
kubectl run pod4 --image=nginx -n ivolve
```

Kubernetes rejected the request because the namespace had already reached its maximum Pod limit.

This confirmed that the ResourceQuota was working correctly.

### 5. Cleanup

After testing, I deleted the three Pods:

```bash
kubectl delete pod pod1 pod2 pod3 -n ivolve
```

I kept the namespace and ResourceQuota configuration.

## Result

Successfully created the `ivolve` namespace and applied a ResourceQuota limiting it to 3 Pods.

The first three Pods were accepted, while the fourth Pod was rejected because it exceeded the quota.

## What I Learned

- Namespaces help organize resources inside a Kubernetes cluster.
- ResourceQuota limits resource consumption within a namespace.
- A Pod quota controls how many Pods can be created in that namespace.
- Kubernetes rejects new Pod creation requests when the quota is exceeded.
- Deleting Pods frees up quota capacity for new Pods.
