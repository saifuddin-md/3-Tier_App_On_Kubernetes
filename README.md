# 3-Tier_App_On_Kubernetes


# 🚀 Deploy Database (mongodb)

Make scripts executable:

```bash
cd database
```

Deploy everything:

```bash
kubectl apply -f
kubectl apply -f 
```

Verify:

```bash
kubectl get pods -A
kubectl get svc -A
kubectl get ingress -A
```




# 🚀 Deploy Application

Make scripts executable:

```bash
chmod +x scripts/*.sh
```

Deploy everything:

```bash
./scripts/deploy.sh
```

Verify:

```bash
kubectl get pods -A
kubectl get svc -A
kubectl get ingress -A
```
