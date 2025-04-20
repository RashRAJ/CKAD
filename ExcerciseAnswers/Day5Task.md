
# Exercise: Pod and Namespace Interaction - Solution

This document provides the solutions to the exercise defined in `Day5-PodAndNamespaces.md`.

## Task Description (Recap)

1.  Create the namespace `ckad-prep`.
2.  In the namespace `ckad-prep`, create a new Pod named `mypod` with the image `nginx:2.3.5`. Expose port `80`.
3.  Identify any issues with creating the container. Write down the root cause of the issue in a file named `pod-error.txt`.
4.  Change the image of the Pod to `nginx:1.15.12`.
5.  List the Pod and ensure that the container is running.
6.  Log into the container and run the `ls` command. Write down the output. Log out of the container.
7.  Retrieve the IP address of the Pod `mypod`.
8.  Run a temporary Pod in the namespace `ckad-prep` using the image `busybox`. Shell into it and run a `wget` command against the `mypod` Pod's IP address on port `80`.
9.  Render the logs of Pod `mypod`.
10. Delete the Pod and the namespace.

## Solution Steps

1.  **Create the namespace `ckad-prep`.**
    ```bash
    kubectl create namespace ckad-prep
    # Expected output: namespace/ckad-prep created
    ```
2.  **In the namespace `ckad-prep`, create a new Pod named `mypod` with the image `nginx:2.3.5`. Expose the port `80`.**
    ```bash
    # Note: Using --port with kubectl run for Pods is deprecated and might not work in newer versions.
    # It's better practice to define ports in a YAML manifest.
    # However, for the exam, if 'kubectl run' is used:
    kubectl run mypod --image=nginx:2.3.5 --restart=Never --port 80 -n ckad-prep # Port exposure might need YAML
    # Expected output: pod/mypod created (but it might fail to run)
    ```
3.  **Identify the issue with creating the container. Write down the root cause of issue in a file named `pod-error.txt`.**
    ```bash
    # Check pod status
    kubectl get pods -n ckad-prep
    # NAME    READY   STATUS         RESTARTS   AGE
    # mypod   0/1     ErrImagePull   0          15s

    # Describe the pod to see events
    kubectl describe pod mypod -n ckad-prep
    # ... Look for Events section ...
    # Normal   Pulling    25s   kubelet            Pulling image "nginx:2.3.5"
    # Warning  Failed     25s   kubelet            Failed to pull image "nginx:2.3.5": rpc error: code = Unknown desc = Error response from daemon: manifest for nginx:2.3.5 not found: manifest unknown: manifest unknown
    # Warning  Failed     25s   kubelet            Error: ErrImagePull
    # Normal   BackOff    12s   kubelet            Back-off pulling image "nginx:2.3.5"
    # Warning  Failed     12s   kubelet            Error: ImagePullBackOff

    # Create the error file
    echo "The Pod status shows 'ErrImagePull' or 'ImagePullBackOff'. The 'kubectl describe' events indicate that the image tag 'nginx:2.3.5' was not found in the registry." > pod-error.txt
    ```
4.  **Change the image of the Pod to `nginx:1.15.12`.**
    ```bash
    # Option 1: Using kubectl set image
    kubectl set image pod/mypod mypod=nginx:1.15.12 -n ckad-prep
    # pod/mypod image updated

    # Option 2: Using kubectl edit
    # kubectl edit pod mypod -n ckad-prep
    # (Manually change spec.containers[0].image to nginx:1.15.12 in the editor)
    ```

5.  **List the Pod and ensure that the container is running.**
    ```bash
    kubectl get pods -n ckad-prep
    # Wait a few moments for the image pull if necessary
    # Expected output:
    # NAME    READY   STATUS    RESTARTS   AGE
    # mypod   1/1     Running   0          3m44s
    ```
6.  **Log into the container and run the `ls` command. Write down the output. Log out of the container.**
    ```bash
    kubectl exec -it mypod -n ckad-prep -- /bin/sh
    # Prompt changes to: /#

    # Run ls inside the container
    ls
    # Example Output:
    # bin   docker-entrypoint.d   etc   lib    media  opt   root  sbin  sys  usr
    # boot  docker-entrypoint.sh  home  lib64  mnt    proc  run   srv   tmp  var

    # Exit the container shell
    exit
    ```

7.  **Retrieve the IP address of the Pod `mypod`.**
    ```bash
    # Option 1: Using -o wide
    kubectl get pod mypod -n ckad-prep -o wide
    # NAME    READY   STATUS    RESTARTS   AGE     IP            NODE                 NOMINATED NODE   READINESS GATES
    # mypod   1/1     Running   0          5m10s   10.244.0.12   kind-control-plane   <none>           <none>

    # Option 2: Using describe and grep (less reliable if output format changes)
    # kubectl describe pod mypod -n ckad-prep | grep IP:
    # IP:               10.244.0.12
    # IPs:
    #  IP:  10.244.0.12
    ```
8.  **Run a temporary Pod in the namespace `ckad-prep` using the image `busybox`. Shell into it and run a `wget` command against the `mypod` Pod's IP address on port `80`.**
    ```bash
    # Replace 10.244.0.12 with the actual IP obtained in the previous step
    kubectl run temp-busybox --image=busybox --rm -it --restart=Never -n ckad-prep -- wget -O- 10.244.0.12:80

    # Expected output (includes Nginx welcome page HTML):
    # Connecting to 10.244.0.12:80 (10.244.0.12:80)
    # <!DOCTYPE html>
    # <html>
    # <head>
    # <title>Welcome to nginx!</title>
    # ...
    # </html>
    # -                    100% |********************************|   612  0:00:00 ETA
    # pod "temp-busybox" deleted
    ```

9.  **Render the logs of Pod `mypod`.**
    ```bash
    kubectl logs mypod -n ckad-prep
    # Expected output (shows the wget request from the temporary busybox pod):
    # /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
    # /docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
    # /docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
    # 10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
    # 10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
    # /docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
    # /docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
    # /docker-entrypoint.sh: Configuration complete; ready for start up
    # 10.244.0.14 - - [20/Apr/2025:14:44:35 +0000] "GET / HTTP/1.1" 200 612 "-" "Wget" "-"
    ```
10. **Delete the Pod and the namespace.**
    ```bash
    kubectl delete pod mypod -n ckad-prep
    # Expected output: pod "mypod" deleted

    kubectl delete namespace ckad-prep
    # Expected output: namespace "ckad-prep" deleted
    ```
