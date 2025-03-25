# Day 3: Kubernetes Services

**Goal**: Learn about service discovery and exposing applications.
**Task**: Create ClusterIP and NodePort services for Nginx deployment.
## Specifically:
- Create a ClusterIP service for internal access
- Create a NodePort service for external access
- Debug common service configuration errors

## Files Created

### 1. nginx-clusterip-service.yaml
ClusterIP service configuration demonstrating:
- Internal service discovery
- Label selectors
- Port mapping

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  type: ClusterIP
  selector:
    app.kubernates.io/name: raj-nginx
  ports:
    - port: 80
      targetPort: 80
```

### 2. nginx-nodeport-service.yaml
NodePort service configuration demonstrating:
- External access pattern
- Static port assignment
- Service type configuration

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app.kubernates.io/name: raj-nginx
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30007
```

## Key Commands Learned

```bash
# Create services
kubectl apply -f nginx-clusterip-service.yaml
kubectl apply -f nginx-nodeport-service.yaml

# Check service status
kubectl get services
kubectl describe service nginx

# Debug service endpoints
kubectl get endpoints
```

## Challenges Encountered & Solutions

1. **API Version Error**
   ```
   error: resource mapping not found for name: "nginx-service" namespace: "" from "nginx-clusterip-service.yaml": no matches for kind "Service" in version "apps/v1"
   ```
   **Learning**: Services use `apiVersion: v1`, not `apps/v1`

2. **Field Name Errors**
   ```
   Error from server (BadRequest): error when creating "nginx-clusterip-service.yaml": Service in version "v1" cannot be handled as a Service: strict decoding error: unknown field "spec.ports[0].ports", unknown field "spec.ports[0].targetport"
   ```
   **Learning**: 
   - Kubernetes is case-sensitive (`targetPort` not `targetport`)
   - Correct field names are critical

3. **Selector Issues**
   ```
   Endpoints: <none>
   ```
   **Learning**: 
   - Selector must match pod labels exactly
   - Typo found: "kubernates" should be "kubernetes"
   - Verify with `kubectl get pods --show-labels`

## Best Practices Learned

1. **Service Configuration**
   - Always verify API version
   - Double-check field names and case sensitivity
   - Ensure selectors match pod labels exactly

2. **Debugging**
   - Use `kubectl describe` to inspect service details
   - Check endpoints to verify pod selection
   - Validate YAML formatting before applying

3. **Service Types**
   - ClusterIP for internal communication
   - NodePort for external access (development only)
   - Consider LoadBalancer for production

## Next Steps
- [ ] Explore Ingress for advanced routing
- [ ] Learn about LoadBalancer services
- [ ] Practice service discovery patterns
- [ ] Understand DNS in Kubernetes
- [ ] Study network policies

## Resources
- [Kubernetes Services](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Service Types](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types)
- [Debugging Services](https://kubernetes.io/docs/tasks/debug/debug-application/debug-service/)
