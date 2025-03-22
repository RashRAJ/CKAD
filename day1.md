## Day 1: Pods and Multi-Container Pods

**Goal**: To understand pod lifecycle, resource requests/limits, and multi-container patterns.
**Task**: Deploy pods with different configurations.
## Specifically:
- Deploy a single-container Nginx pod.
- Deploy a pod with two containers: Nginx and a sidecar container that logs the Nginx access logs to stdout.
- Experiment with setting resource requests and limits for the containers.

# Resources:
- Kubernetes documentation: https://kubernetes.io/docs/concepts/workloads/pods/

# Kubernetes Learning - Day 1

## Overview
This repository contains basic Kubernetes Pod configurations exploring fundamental concepts including:
- Basic Pod creation
- Multi-container Pods
- Volume sharing
- Init containers

## Files

### 1. nginx-pod.yml
Basic nginx Pod configuration demonstrating:
- Simple Pod creation
- Container port exposure
- Proper YAML formatting for Kubernetes manifests


### 2. nginx-sidecar-pod.yaml
More advanced Pod configuration showcasing:
- Multi-container pattern with init container
- Volume sharing between containers
- Sidecar logging pattern



## Key Learnings

1. **Pod Basics**
   - Pods are the smallest deployable units in Kubernetes
   - Field names in Kubernetes YAML are case-sensitive (e.g., `containers` not `Containers`)
   - Pod specifications have strict validation rules

2. **Pod Immutability**
   - Most Pod fields are immutable after creation
   - Only certain fields can be updated (e.g., container images, tolerations)
   - For updating other fields, Pods must be recreated

3. **Multi-container Patterns**
   - Pods can have multiple containers sharing resources
   - Init containers run before app containers
   - Volumes can be shared between containers in a Pod

## Common Commands

### Essential Commands Used
```bash
# Create/Update resources
kubectl apply -f <filename>

# Delete resources when updates not allowed
kubectl delete pod <pod-name>

# View Pod status
kubectl get pods

# Debug Pod issues
kubectl describe pod <pod-name>
```

### Best Practices Learned
1. Always validate YAML case sensitivity
2. Use `kubectl describe` for debugging pod issues
3. Consider pod immutability when planning updates
4. Use appropriate container patterns (sidecar vs init container)
5. Implement shared volumes correctly for container communication

### Next Steps
- [ ] Explore Deployments for better update strategies
- [ ] Study ConfigMaps and Secrets for configuration management
- [ ] Learn about health checks and probe configurations
- [ ] Practice debugging techniques for multi-container pods
- [ ] Understand resource quotas and limits

### Resources
- [Kubernetes Pod Concepts](https://kubernetes.io/docs/concepts/workloads/pods/)
- [Multi-Container Pod Patterns](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
- [Pod Lifecycle Documentation](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)



