# Day 2: Deployments and ReplicaSets

**Goal**: Learn about declarative application updates and scaling.
**Task**: Create rolling updates and rollbacks.
## Specifically:
- Create a Deployment for an Nginx application.
- Scale the Deployment to multiple replicas.
- Perform a rolling update to change the Nginx version.
- Perform a rollback to the previous version.



## Objectives
- Understanding Deployments and their benefits over raw Pods
- Learning deployment strategies (Rolling Update vs Recreate)
- Managing application updates and rollbacks
- Scaling applications

## Files Created

### 1. nginx-deployment.yaml
Basic Deployment configuration demonstrating:
- ReplicaSet management
- Update strategy configuration
- Label selectors
- Pod template specification

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

## Key Commands Learned

```bash
# Create and manage deployments
kubectl create deployment nginx --image=nginx:1.14.2
kubectl apply -f nginx-deployment.yaml

# Scale deployments
kubectl scale deployment nginx-deployment --replicas=5

# Update image
kubectl set image deployment/nginx-deployment nginx=nginx:1.15.0

# Check rollout status
kubectl rollout status deployment/nginx-deployment

# View rollout history
kubectl rollout history deployment/nginx-deployment

# Rollback to previous version
kubectl rollout undo deployment/nginx-deployment

# Pause/Resume rollout
kubectl rollout pause deployment/nginx-deployment
kubectl rollout resume deployment/nginx-deployment
```

## Challenges Encountered & Solutions

1. **Selector Configuration Error**
   ```yaml
   # Error
   The Deployment "nginx-deployment" is invalid: 
   spec.selector: Required value
   ```
   **Learning**: Deployments require a selector to match Pod labels. Always specify both selector and Pod template labels.

2. **Rolling Update Issues**
   ```yaml
   # Error
   Waiting for deployment "nginx-deployment" rollout to finish: 
   1 old replicas are pending termination
   ```
   **Learning**: 
   - Default rolling update strategy might be too aggressive
   - Configure `maxSurge` and `maxUnavailable` for better control
   ```yaml
   spec:
     strategy:
       rollingUpdate:
         maxSurge: 1
         maxUnavailable: 0
   ```

3. **Image Pull Errors**
   ```
   Error: ErrImagePull / ImagePullBackOff
   ```
   **Learning**:
   - Always verify image names and tags
   - Check container registry accessibility
   - Use image pull secrets when needed

## Best Practices Learned

1. **Deployment Strategy**
   - Use Rolling Update for zero-downtime deployments
   - Configure appropriate readiness probes
   - Set resource requests/limits

2. **Labels and Selectors**
   - Use meaningful labels
   - Maintain consistency between selector and template labels
   - Add additional labels for better organization

3. **Version Control**
   - Keep track of deployment versions
   - Use meaningful image tags
   - Document rollback procedures

## Next Steps
- [ ] Explore Services and Ingress
- [ ] Study ConfigMaps and Secrets integration
- [ ] Learn about StatefulSets
- [ ] Practice advanced rollout strategies
- [ ] Understand horizontal pod autoscaling

## Resources
- [Kubernetes Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Rolling Update Strategy](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-update-deployment)
- [Scaling Applications](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment)
