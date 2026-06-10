
**Apply:**
```bash
kubectl apply -f platform/cluster-autoscaler/rbac.yaml

kubectl apply -f platform/cluster-autoscaler/deployment.yaml
```


**Verify:**
```bash
kubectl get pods -n kube-system | grep cluster-autoscaler

kubectl logs -n kube-system deployment/cluster-autoscaler
```

**Before applying, make sure:**

- IRSA IAM role exists.
- OIDC provider is associated with your EKS cluster.
- Your node group's ASG has the required Cluster Autoscaler tags.
- The autoscaler version matches your Kubernetes/EKS version.
