# 🚀 EKS 3-Tier DevOps Platform

Production-style Kubernetes project deployed on Amazon EKS using raw Kubernetes manifests, Jenkins CI/CD, autoscaling, ingress management, monitoring, and operational automation.

---

# 📌 Project Overview

This repository demonstrates deployment and management of a cloud-native 3-tier application architecture on Amazon EKS.

The project follows a realistic Kubernetes repository structure used in modern DevOps environments with clear separation between:

- Application workloads
- Stateful services
- Platform tooling
- External dependencies
- Operational scripts

The platform includes:

- Frontend & Backend microservices
- Database layer with persistent storage
- NGINX Ingress Controller
- cert-manager
- Metrics Server
- Horizontal Pod Autoscaler (HPA)
- Cluster Autoscaler
- Monitoring stack
- Jenkins CI/CD integration

---

# 🏗️ Architecture

```text
                    Internet
                        │
                        ▼
            NGINX Ingress Controller
                        │
        ┌───────────────┴───────────────┐
        ▼                               ▼
   Frontend Service                Backend Service
        │                               │
        └───────────────┬───────────────┘
                        ▼
                    Database
                        │
                        ▼
                 Persistent Volume
```

---

# 📂 Repository Structure

```bash
kubernetes/

├── app/                             # Application workloads
│   ├── namespace.yaml
│   │
│   ├── frontend/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── ingress.yaml
│   │   └── hpa.yaml
│   │
│   └── backend/
│       ├── deployment.yaml
│       ├── service.yaml
│       └── hpa.yaml
│
├── database/                        # Stateful components
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── pvc.yaml
│   └── secret.yaml
│
├── platform/                        # Cluster-level configs
│   └── monitoring/
│       ├── prometheus.yaml
│       └── grafana.yaml
│
├── external/
│   ├── cluster-autoscaler.md                      # Third-party installations
│   ├── ingress-&-cert-manager.md
│   └── metrics-server.md
│
├── scripts/
│   ├── deploy.sh
│   ├── destroy.sh
│   └── autoscaler.sh
│
├── Makefile
│
└── README.md
```

---

# 🧠 Why This Structure Is Strong

## ✅ Repository owns:

- Application manifests
- Database configuration
- Autoscaling configuration
- Monitoring setup
- Operational scripts

## ✅ External vendors own:

- NGINX Ingress Controller
- cert-manager
- AWS Load Balancer Controller

This avoids storing huge third-party manifests inside the repository and keeps ownership boundaries clean.

---

# ⚙️ Prerequisites

Install the following tools:

- AWS CLI
- kubectl
- eksctl
- Helm
- Docker
- Jenkins

Verify:

```bash
kubectl version --client
helm version
aws --version
```

---

# ☁️ Create EKS Cluster

Create EKS cluster manually using eksctl:

```bash
eksctl create cluster \
  --name my-eks-cluster \
  --region ap-south-1 \
  --nodegroup-name workers \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 3 \
  --managed
```

Update kubeconfig:

```bash
aws eks update-kubeconfig \
  --region ap-south-1 \
  --name my-eks-cluster
```

Verify cluster:

```bash
kubectl get nodes
```

---

# 🌐 Install NGINX Ingress Controller

File:

```text
external/ingress-nginx.md
```

Install:

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace
```

Verify:

```bash
kubectl get pods -n ingress-nginx
```

---

# 🔐 Install cert-manager

File:

```text
external/cert-manager.md
```

Install:

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update

helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set installCRDs=true
```

Verify:

```bash
kubectl get pods -n cert-manager
```

---

# 📈 Deploy Metrics Server

```bash
kubectl apply -f kubernetes/platform/metrics-server/
```

Verify:

```bash
kubectl top nodes
```

---

# ⚡ Deploy Cluster Autoscaler

```bash
kubectl apply -f kubernetes/platform/cluster-autoscaler/
```

Verify:

```bash
kubectl get pods -n kube-system
```

---

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

---

# 📊 Horizontal Pod Autoscaler

Check HPA:

```bash
kubectl get hpa
```

Scaling flow:

```text
Traffic Spike
      ↓
CPU Usage Increase
      ↓
HPA Adds Pods
      ↓
Insufficient Node Capacity
      ↓
Cluster Autoscaler Adds EC2 Nodes
```

---

# 🔥 Monitoring Stack

Deploy monitoring:

```bash
kubectl apply -f kubernetes/platform/monitoring/
```

Components:

| Component | Purpose |
|---|---|
| Prometheus | Metrics collection |
| Grafana | Visualization dashboards |
| Metrics Server | Kubernetes resource metrics |

---

# 📜 Operational Scripts

The repository includes reusable operational shell scripts for deployment and lifecycle management.

Repository:

```bash
scripts/
├── deploy.sh
├── destroy.sh
└── autoscaler.sh
```

---

## 🚀 deploy.sh

Deploys the complete platform stack.

```bash
#!/bin/bash

set -e

echo "🚀 Creating namespace..."
kubectl apply -f kubernetes/app/namespace.yaml

echo "📦 Deploying database..."
kubectl apply -f kubernetes/database/

echo "📈 Deploying Metrics Server..."
kubectl apply -f kubernetes/platform/metrics-server/

echo "⚡ Deploying Cluster Autoscaler..."
kubectl apply -f kubernetes/platform/cluster-autoscaler/

echo "🌐 Deploying frontend..."
kubectl apply -f kubernetes/app/frontend/

echo "🔧 Deploying backend..."
kubectl apply -f kubernetes/app/backend/

echo "✅ Deployment complete"
```

Run:

```bash
./scripts/deploy.sh
```

---

## 💥 destroy.sh

Deletes all workloads and platform resources.

```bash
#!/bin/bash

set -e

echo "🗑 Removing application workloads..."
kubectl delete -f kubernetes/app/ --ignore-not-found

echo "🗑 Removing database..."
kubectl delete -f kubernetes/database/ --ignore-not-found

echo "🗑 Removing platform components..."
kubectl delete -f kubernetes/platform/ --ignore-not-found

echo "✅ Cleanup complete"
```

Run:

```bash
./scripts/destroy.sh
```

---

## 📈 autoscaler.sh

Deploys or updates Cluster Autoscaler.

```bash
#!/bin/bash

set -e

echo "📈 Deploying Cluster Autoscaler..."

kubectl apply -f kubernetes/platform/cluster-autoscaler/

echo "✅ Cluster Autoscaler deployed"
```

Run:

```bash
./scripts/autoscaler.sh
```

---

# 🛠️ Makefile Automation

The project uses a Makefile to simplify common operational commands.

Example Makefile:

```makefile
install-ingress:
	helm install ingress-nginx ingress-nginx/ingress-nginx \
	--namespace ingress-nginx \
	--create-namespace

install-cert-manager:
	helm install cert-manager jetstack/cert-manager \
	--namespace cert-manager \
	--create-namespace \
	--set installCRDs=true

deploy:
	chmod +x scripts/*.sh
	./scripts/deploy.sh

autoscaler:
	./scripts/autoscaler.sh

monitoring:
	kubectl apply -f kubernetes/platform/monitoring/

cleanup:
	./scripts/destroy.sh
```

Usage:

```bash
make deploy
```

Other commands:

```bash
make autoscaler
make monitoring
make cleanup
```

---

# 🔁 Deployment Flow

```text
1. Create EKS Cluster
2. Install ingress-nginx
3. Install cert-manager
4. Deploy metrics-server
5. Deploy cluster-autoscaler
6. Deploy database
7. Deploy frontend & backend
8. Verify ingress and scaling
```

---

# 🧹 Cleanup

Destroy workloads:

```bash
./scripts/destroy.sh
```

Delete EKS cluster:

```bash
eksctl delete cluster \
  --name my-eks-cluster \
  --region ap-south-1
```

---

# 🧠 Kubernetes Concepts Used

- Deployments
- Services
- Ingress
- Horizontal Pod Autoscaler
- PersistentVolumeClaims
- Secrets
- RBAC
- Namespaces
- Metrics Server
- Cluster Autoscaler
- Resource Requests & Limits

---

# 🎯 Resume Description

> Built and deployed a production-style 3-tier application platform on Amazon EKS using Kubernetes, Jenkins CI/CD, ingress management, autoscaling, monitoring, and operational automation.

---

# 🚀 Future Improvements

- Helm migration
- ArgoCD GitOps
- External Secrets Operator
- Loki logging stack
- KEDA event-driven autoscaling
- AWS Load Balancer Controller

---

# 📜 License

MIT License

