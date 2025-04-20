# Kubernetes Pods and Namespaces

## Pods

A **Pod** is the smallest deployable unit in Kubernetes. It represents a single instance of a running process in your cluster. A Pod can contain one or more containers (like Docker containers), which share storage and network resources, and a specification for how to run the containers. Pods encapsulate application containers, storage resources, a unique network IP, and options that govern how the container(s) should run.

## Namespaces

A **Namespace** provides a mechanism for isolating groups of resources within a single cluster. Namespaces are intended for use in environments with many users spread across multiple teams or projects. They provide a scope for names: names of resources need to be unique within a namespace, but not across namespaces. Namespaces cannot be nested inside one another and each Kubernetes resource can only be in one namespace.

## Creating a Pod with Imperative Commands

You can create a Pod directly using the `kubectl run` command.

```bash
# Create a Pod named 'hazelcast' using the 'hazelcast/hazelcast' image
# --restart=Never indicates this is a standalone Pod, not managed by a Deployment/ReplicaSet
# --port exposes the container port
# --env sets environment variables
# --labels applies labels to the Pod
kubectl run hazelcast --image=hazelcast/hazelcast --restart=Never --port=5701 --env="DNS_DOMAIN=cluster" --labels="app=hazelcast,env=prod"

# Expected output:
# pod/hazelcast created
```

## Pod YAML Manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hazelcast
  labels:
  name: hazelcast
  labels:
    app: hazelcast
    env: prod
spec:
  containers:
  - name: hazelcast # Name of the container within the Pod
    image: hazelcast/hazelcast # Docker image to use
    ports:
    - containerPort: 5701 # Port the container exposes
    env:
    - name: DNS_DOMAIN # Environment variable name
      value: cluster   # Environment variable value
```

## Listing Pods

```bash
# Get all pods in the current namespace
kubectl get pods

# Example Output:
# NAME                         READY   STATUS    RESTARTS       AGE
# hazelcast                    1/1     Running   0              4m13s

# Get a specific pod by name
kubectl get pod hazelcast

# Example Output:
# NAME        READY   STATUS    RESTARTS   AGE
# hazelcast   1/1     Running   0          4m47s
```

## Describing a Pod

```bash
# Get detailed information about a specific pod, including events
kubectl describe pod hazelcast

# Example Output (abbreviated):
# Name:             hazelcast
# Namespace:        default
# Priority:         0
# Service Account:  default
# Node:             kind-control-plane/172.18.0.2
# Start Time:       Sun, 20 Apr 2025 15:00:07 +0100
# Labels:           app=hazelcast
#                   env=prod
# Annotations:      <none>
# Status:           Running
# IP:               10.244.0.10
# IPs:
#   IP:  10.244.0.10
# ...
# Events:
#   Type    Reason     Age    From               Message
#   ----    ------     ----   ----               -------
#   Normal  Scheduled  6m43s  default-scheduler  Successfully assigned default/hazelcast to kind-control-plane
#   Normal  Pulling    6m43s  kubelet            Pulling image "hazelcast/hazelcast"
#   Normal  Pulled     5m43s  kubelet            Successfully pulled image "hazelcast/hazelcast" in 59.911s (59.911s including waiting). Image size: 551578413 bytes.
#   Normal  Created    5m43s  kubelet            Created container hazelcast
#   Normal  Started    5m42s  kubelet            Started container hazelcast
```

## Getting Pod Logs

```bash
# Fetch logs from the 'hazelcast' pod
kubectl logs hazelcast

# Example Output (abbreviated):
# ########################################
# # JAVA=/usr/bin/java
# # JAVA_OPTS=--add-modules java.se --add-exports java.base/jdk.internal.ref=ALL-UNNAMED --add-opens java.base/
# ########################################
#
#     o    o     o     o---o   o--o o      o---o     o     o----o o--o--o
#     |    |    / \       /         |     /         / \    |         |
#     o----o       o     o   o----o |    o             o   o----o    |
#     |    |  *     \   /           |     \       *     \       |    |
#     o    o *       o o---o   o--o o----o o---o *       o o----o    o
# ...
```

## Executing Commands in a Container

``` bash
# Execute an interactive shell (-it) inside the 'hazelcast' pod
kubectl exec -it hazelcast -- /bin/sh

# Example prompt inside the container:
# /opt/hazelcast $

# Execute a specific command ('env') directly in the container without an interactive shell
kubectl exec hazelcast -- env

# Example Output (abbreviated):
# PATH=/opt/hazelcast/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
# HOSTNAME=hazelcast
# HZ_HOME=/opt/hazelcast
# ...
```

## Creating a Temporary Pod for Debugging

```bash
#the -rm flag removes the pod after executing the comand
# Run a 'busybox' pod, execute 'wget' against another pod's IP, then automatically remove the pod (--rm)
# The '-it' flags provide an interactive terminal
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget 10.244.0.10

# Example Output:
# Connecting to 10.244.0.10 (10.244.0.10:80)
# ... (wget output) ...
# pod "busybox" deleted
```

## Declaring Commands and Arguments in a Pod Manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: spring-boot-app
spec:
  container:
  - image: bmuschko/spring-boot-app:1.5.3
    name: spring-boot-app
    # Overrides the default container entrypoint
    command: ["/bin/sh"]
    # Arguments passed to the command
    args: ["-c", "while true; do date; sleep 10; done"]
```

## Deleting a Pod

```bash
# Imperative approach: delete by name
kubectl delete pod hazelcast

# Declarative approach: delete using the manifest file
kubectl delete -f pod.yaml

# Force delete a pod immediately (use with caution, bypasses graceful termination)
# The --now flag is deprecated, use --force --grace-period=0 instead
kubectl delete pod hazelcast --force --grace-period=0
```

## Creating a Namespace with Imperative Commands
```bash
# Create a namespace named 'ckad'
kubectl create namespace ckad

# Expected output:
# namespace/ckad created

# List all namespaces in the cluster
kubectl get namespaces
```

## Namespace YAML Manifest

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ckad # Name of the namespace
```

## Deleting a Namespace

```bash
# Delete the namespace 'ckad'
kubectl delete namespace ckad

# Expected output:
# namespace/ckad deleted

# Note: Deleting a namespace deletes *all* objects (Pods, Services, Deployments, etc.) within that namespace.
```

## Exercise: Pod and Namespace Interaction

In this exercise, you will practice creating a new Pod within a specific namespace. Once created, you will inspect it, execute commands inside its container, and interact with it from another temporary Pod.

1.  Create the namespace `ckad-prep`.
2.  In the namespace `ckad-prep`, create a new Pod named `mypod` with the image `nginx:2.3.5`. Expose port `80`. *(Note: `nginx:2.3.5` might be an invalid image tag; you may need to use a valid one like `nginx:1.25` or `nginx:latest` if it fails).*
3.  Identify any issues with creating the container (e.g., using `kubectl describe pod mypod -n ckad-prep`). Write down the root cause of the issue in a file named `pod-error.txt`.
4.  If necessary, change the image of the Pod to a working version, for example, `nginx:1.25.3` (use `kubectl edit pod mypod -n ckad-prep` or `kubectl set image pod/mypod nginx=nginx:1.25.3 -n ckad-prep`).
5.  List the Pod in the `ckad-prep` namespace and ensure that the container is running (`kubectl get pods -n ckad-prep`).
6.  Log into the container (`kubectl exec -it mypod -n ckad-prep -- /bin/bash` or `/bin/sh`) and run the `ls` command. Write down the output. Log out of the container (type `exit`).
7.  Retrieve the IP address of the Pod `mypod` (`kubectl get pod mypod -n ckad-prep -o wide` or `kubectl describe pod mypod -n ckad-prep | grep IP:`).
8.  Run a temporary Pod in the namespace `ckad-prep` using the image `busybox`. Shell into it (`kubectl run temp-debug --image=busybox --rm -it --restart=Never -n ckad-prep -- sh`) and run a `wget` command against the `mypod` Pod's IP address on port `80` (e.g., `wget <mypod-ip>:80`). Exit the temporary pod.
9.  Render the logs of Pod `mypod` (`kubectl logs mypod -n ckad-prep`).
10. Delete the Pod (`kubectl delete pod mypod -n ckad-prep`) and the namespace (`kubectl delete namespace ckad-prep`).
