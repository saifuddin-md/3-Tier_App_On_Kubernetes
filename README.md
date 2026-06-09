# 3-Tier_App_On_Kubernetes


# 🚀 Deploy Database (mongodb)

## Step 1 - **Create namespace (rr-app):**

```bash
cd app
kubectl apply -f namespace.yaml
kubectl get ns
```

## Step 2 - **Deploy database (mongodb):**

```bash
kubectl apply -f mongo-secret.yaml
kubectl apply -f mongo-statefulset.yaml
```

**Verify:**

```bash
kubectl get pods -n rr-app
kubectl get svc -n rr-app
kubectl get all -n rr-app
kubectl get pv
kubectl get pvc -n rr-app
kubectl config set-context --current --namespace rr-app  # set default namespace as rr-app
```
## Step 3 — Initialize Replica Set First

```bash
kubectl exec -it mongo-0 -n rr-app -- mongo
```

**Run:**

```sql
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "mongo-0.mongo-svc:27017" },
    { _id: 1, host: "mongo-1.mongo-svc:27017" },
    { _id: 2, host: "mongo-2.mongo-svc:27017" }
  ]
})
```

Wait few seconds.

**Verify:**

```bash
rs.status()
```
**Ensure mongo-0 becomes PRIMARY.**

## Step 4 - Execute **init.js**   (From your local machine:)

```bash
kubectl exec -i mongo-0 -n rr-app -- mongo < init.js
oR
kubectl exec -i mongo-0 -n rr-app -- \
mongo "mongodb://mongo-0.mongo-svc:27017,mongo-1.mongo-svc:27017,mongo-2.mongo-svc:27017/restaurant_reviews?replicaSet=rs0" < init.js
```

## Step 5 - Verify Data

**Open mongo shell:**

```bash
kubectl exec -it mongo-0 -n rr-app -- mongo
```
**Switch DB:**

```bash
use restaurant_reviews
```

**Check collections:**

```bash
show collections
```
**Check data:**

```bash
db.restaurants.find().pretty()
```
**Check reviews:**

```bash
db.reviews.find().pretty()
```
---
## Check mongo-1 receives replicated data

```bash
kubectl exec -it mongo-1 -n rr-app -- mongo   
```
**Switch DB:**

```bash
use restaurant_reviews
```
**Enable reads:**

```bash
rs.slaveOk()
```
**Check collections:**

```bash
show collections
```
**Check data:**

```bash
db.restaurants.find().pretty()
```

**Check reviews:**
```bash
db.reviews.find().pretty()
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
