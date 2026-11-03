# Lab 18 – Node-Wide Pod Management with DaemonSet

## Objective

The goal of this lab is to deploy a monitoring stack in Kubernetes and use a DaemonSet to run Node Exporter on every node.

## What I Did

1. Created a dedicated `monitoring` namespace.
2. Deployed Prometheus to collect monitoring metrics.
3. Deployed Alertmanager for handling alerts.
4. Deployed Grafana for visualizing metrics.
5. Created a DaemonSet for Prometheus Node Exporter.
6. Configured tolerations to allow Node Exporter to run on all three nodes, including the control plane.
7. Verified that one Node Exporter Pod was running on each node.
8. Accessed the Node Exporter metrics endpoint on port `9100`.

## Verification

Checked the DaemonSet:

```bash
kubectl get daemonset -n monitoring
```

Result: 3 desired, 3 ready, and 3 available.

Checked Pod placement:

```bash
kubectl get pods -n monitoring -o wide
```

Confirmed that Node Exporter was running on:

- `k8s-control`
- `k8s-worker1`
- `k8s-worker2`

Tested the metrics endpoint:

```bash
curl http://10.244.126.32:9100/metrics
```

The endpoint successfully returned Prometheus-formatted metrics.

## Screenshots

Add the following screenshots:

- DaemonSet showing 3/3 ready.
- Monitoring Pods running on the cluster nodes.
- Node Exporter metrics output.

## Notes

Prometheus scraping configuration and Grafana dashboard setup were treated as optional bonus tasks and were not completed.

## What I Learned

A DaemonSet automatically ensures that one Pod runs on every eligible node, without manually specifying the replica count. This is especially useful for node-level monitoring agents like Node Exporter.
