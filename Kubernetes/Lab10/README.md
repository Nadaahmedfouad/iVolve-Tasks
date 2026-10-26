# Lab 10: Node Isolation Using Taints and Labels

## Objective

The goal of this lab is to isolate a Kubernetes worker node using a taint and identify it using a label. I also tested how a Pod can be scheduled on the isolated node using a toleration and nodeSelector.

## Steps

### 1. Check the Cluster Nodes

First, I checked that all nodes were running:

```bash
kubectl get nodes -o wide
```

The cluster had one control plane and two worker nodes. All three nodes were Ready.

### 2. Apply a Taint

I applied a taint to `k8s-worker2`:

```bash
kubectl taint nodes k8s-worker2 workload=ivolveApp:NoSchedule
```

This prevents new Pods from being scheduled on the node unless they have a matching toleration.

### 3. Add a Label

I labeled the same worker node:

```bash
kubectl label node k8s-worker2 project=ivolve
```

This label can be used to target the node when scheduling Pods.

### 4. Verify the Configuration

I used the following commands to check the taint and label:

```bash
kubectl describe node k8s-worker2
kubectl get nodes --show-labels
```

Both configurations were applied successfully.

### 5. Test Pod Scheduling

I first created an Nginx Pod with a nodeSelector targeting `project=ivolve`, but without a toleration.

The Pod stayed Pending because the node had a `NoSchedule` taint.

I then recreated the Pod using the following YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-isolation
spec:
  nodeSelector:
    project: ivolve

  tolerations:
    - key: "workload"
      operator: "Equal"
      value: "ivolveApp"
      effect: "NoSchedule"

  containers:
    - name: nginx
      image: nginx
```

Applied it using:

```bash
kubectl apply -f test-isolation.yaml
```

Then checked the Pod:

```bash
kubectl get pods -o wide
```

The Pod was Running successfully on `k8s-worker2`.

Finally, I deleted the test Pod:

```bash
kubectl delete pod test-isolation
```

## Result

The taint and label were successfully applied to `k8s-worker2`. The scheduling test also confirmed that a Pod needs a matching toleration to run on the tainted node.

## What I Learned

- Taints restrict which Pods can be scheduled on a node.
- Tolerations allow Pods to be scheduled on nodes with matching taints.
- Labels help identify nodes.
- nodeSelector allows Pods to target specific nodes based on their labels.
- Using these features together helps control workload placement in Kubernetes.
