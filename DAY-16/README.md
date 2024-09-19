# Kubernetes Memory Resource Assignment and Limits

## Task Overview

This guide covers the following steps:
1. Creating a namespace `mem-example`.
2. Installing Metrics Server for resource monitoring.
3. Assigning memory requests and limits to a container.
4. Testing memory limits by exceeding them.
5. Specifying a memory request that is too big for your nodes.

### Step 1: Create a New Namespace

Create a namespace for all memory resource tasks:

```
kubectl create namespace mem-example
```
### Step 2: Install Metrics Server
Install the Metrics Server to monitor node and pod resource usage.

```
git clone https://github.com/kubernetes-sigs/metrics-server.git
kubectl apply -f metrics-server/deploy/kubernetes/metrics-server.yaml
```

Check that the Metrics Server is running:

```
kubectl get apiservice v1beta1.metrics.k8s.io -o yaml
kubectl top nodes
```
### Step 3: Assign Memory Requests and Limits

Create a pod with defined memory requests and limits:

```
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo
  namespace: mem-example
spec:
  containers:
  - name: memory-demo-ctr
    image: polinux/stress
    resources:
      requests:
        memory: "64Mi"
      limits:
        memory: "128Mi"
    args:
    - "--vm"
    - "1"
    - "--vm-bytes"
    - "128M"
    - "--vm-hang"
    - "1"
```
Apply the pod configuration:
```
kubectl apply -f memory-request-limit.yaml
```
### Step 4: Exceed a Container's Memory Limit

Test what happens when the container exceeds its memory limit.

````
apiVersion: v1
kind: Pod
metadata:
  name: memory-limit-exceed
  namespace: mem-example
spec:
  containers:
  - name: memory-limit-ctr
    image: polinux/stress
    resources:
      requests:
        memory: "64Mi"
      limits:
        memory: "128Mi"
    args:
    - "--vm"
    - "1"
    - "--vm-bytes"
    - "256M"  # Exceeding the limit of 128Mi
    - "--vm-hang"
    - "1"
```
Apply the pod configuration and observe:

```
kubectl apply -f memory-limit-exceed.yaml
kubectl describe pod memory-limit-exceed -n mem-example
kubectl get events --namespace mem-example
```
The pod should be evicted with an OOMKilled event.

### Step 5: Specify a Memory Request That Is Too Big for Your Nodes

Create a pod that requests more memory than the nodes can handle:

```
apiVersion: v1
kind: Pod
metadata:
  name: memory-too-large
  namespace: mem-example
spec:
  containers:
  - name: memory-large-ctr
    image: nginx
    resources:
      requests:
        memory: "64Gi"  # This value is too large for most nodes
      limits:
        memory: "64Gi"
```
Apply the pod configuration and check its status:

```
kubectl apply -f memory-too-large.yaml
kubectl describe pod memory-too-large -n mem-example
```

The pod will stay in the Pending state with an event like 0/1 nodes are available: 1 Insufficient memory.




