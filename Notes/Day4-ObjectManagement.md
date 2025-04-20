# Kubernetes Primitives

Kubernetes primitives are the basic building blocks for creating and operating an application within Kubernetes.

**General Primitive Structure**:

Most Kubernetes objects follow a general structure:

```yaml
apiVersion: group/version # Defines the structure and API version
kind: ObjectType         # Defines the type of primitive (e.g., Pod, Service, Deployment)
metadata:                # Describes high-level information about the object (name, labels, namespace)
  name: object-name
  # ... other metadata fields
spec:                    # Desired state of the object (configuration provided by the user)
  # ... object-specific configuration
status:                  # Actual state of the object (managed by Kubernetes, read-only)
  # ... current state information
```

**Key Sections Explained:**

*   **`apiVersion`**: Defines the structure of the primitive and the Kubernetes API version used to validate the data.
*   **`kind`**: Defines the type of primitive (e.g., Pod, Deployment, Service).
*   **`metadata`**: Contains high-level information identifying the object, like its name, namespace, and labels.
*   **`spec`**: Describes the desired state of the object – what you want Kubernetes to achieve.
*   **`status`**: Describes the actual state of the object as observed by the Kubernetes system. This field is managed by Kubernetes and is empty if the object hasn't been created or processed yet.

## Managing Objects with `kubectl`

The `kubectl` command-line tool is used to interact with the Kubernetes API and manage objects. The general syntax is:

```bash
kubectl [command] [TYPE] [NAME] [flags]
```

**Example:**

```bash
# Get information about the pod named 'app' and output it in YAML format
kubectl get pod app -o yaml
```

## Imperative Object Management

This approach involves directly telling Kubernetes what action to perform on an object using commands like `kubectl create`, `kubectl run`, `kubectl edit`, etc.

```bash
# Create an object (e.g., a Namespace)
kubectl create namespace ckad

# Create a Pod object named 'nginx' using the 'nginx' image in the 'ckad' namespace
kubectl run nginx --image=nginx -n ckad

# Open an editor to modify the live 'nginx' pod object in the 'ckad' namespace
kubectl edit pod nginx -n ckad
```

> **Note:** While `kubectl run` can create Pods, it's generally recommended to use **Deployments** for managing application Pods in production environments due to their lifecycle management features.

## Declarative Object Management

This approach involves defining the desired state of objects in manifest files (usually YAML) and letting Kubernetes figure out how to achieve that state using `kubectl apply`.

```bash
# Create objects defined in the file if they don't exist yet
kubectl create -f nginx-deployment.yaml

# Create objects or update them if they already exist to match the file's definition
kubectl apply -f nginx-deployment.yaml

# Delete objects defined in the file
kubectl delete -f nginx-deployment.yaml
```

## Hybrid Approach

You can generate a basic YAML manifest using `kubectl` and then modify it before applying.

```bash
# Generate the YAML for creating an nginx pod without actually creating it on the cluster
# The --dry-run=client flag ensures the operation happens locally
kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx-pod.yaml
```

# Now you have a starting point for your pod manifest. Edit it as needed:
```bash
vim nginx-pod.yaml
```

# Apply the configuration from the modified file
```bash
kubectl apply -f nginx-pod.yaml
```

## Key Points

*   The **imperative** approach (`kubectl create`, `run`, `edit`) can be faster for simple, one-off tasks (useful during exams) but isn't suitable for all object types or complex configurations, and it's harder to track changes.
*   The **declarative** approach (`kubectl apply -f <file>`) is the recommended best practice for managing applications, as it allows for version control, easier updates, and management of more complex configurations (e.g., ConfigMaps, Secrets).
*   When deleting objects quickly (e.g., namespaces), you can use the `--force` flag combined with `--grace-period=0` to bypass the default graceful termination period. **Use with caution.**

```bash
# Example: Force delete a namespace immediately (use carefully!)
kubectl delete namespace ckad --force --grace-period=0
