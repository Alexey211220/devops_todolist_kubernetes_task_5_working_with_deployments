# Kubernetes Deployment Instructions

## Prerequisites

Before deploying the application, make sure that:

- a Kubernetes cluster is running;
- `kubectl` is installed and connected to the cluster;
- the `mateapp` namespace manifest is present;
- the application image is available in the container registry.

## Deploy the application

Apply all Kubernetes manifests from the `.infrastructure` directory:

```bash
kubectl apply -f .infrastructure/
```

Verify the created resources:

```bash
kubectl get deployments -n mateapp
kubectl get pods -n mateapp
kubectl get hpa -n mateapp
kubectl get services -n mateapp
```

The Deployment starts with two replicas.

## Resource requests and limits

Each application container uses the following resource configuration:

- CPU request: `250m`
- CPU limit: `500m`
- Memory request: `64Mi`
- Memory limit: `128Mi`

These values are reasonable initial settings for a small Django application. Requests reserve enough resources for normal operation, while limits allow additional capacity during short load increases. The limits are twice the requests, giving the container room to handle temporary traffic spikes.

In a production environment, these values should be adjusted after monitoring the real CPU and memory usage of the application.

## Horizontal Pod Autoscaler

The HPA keeps between 2 and 5 application pods.

- `minReplicas: 2` keeps two pods available during normal operation.
- `maxReplicas: 5` prevents the application from consuming unlimited cluster resources.
- CPU and memory targets are set to `70%` of the configured resource requests.

The 70% threshold leaves some available capacity before the pods become fully loaded. More accurate thresholds should be selected after load testing and observing real application metrics.

## RollingUpdate strategy

The Deployment uses the `RollingUpdate` strategy:

- `maxSurge: 1`
- `maxUnavailable: 0`

`maxSurge: 1` allows Kubernetes to create one additional pod during an update. `maxUnavailable: 0` ensures that the number of available pods does not fall below the desired replica count.

With two replicas, Kubernetes creates a new pod, waits until it becomes ready, and only then removes an old pod. This keeps the application available during updates.

## Access the application

The application is exposed through a NodePort Service.

Check the Service and its assigned NodePort:

```bash
kubectl get services -n mateapp
```

Check the cluster node IP:

```bash
kubectl get nodes -o wide
```

Open the application using:

```text
http://<NODE_IP>:30080
```