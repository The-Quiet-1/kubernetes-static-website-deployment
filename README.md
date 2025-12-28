# 🚀 Kubernetes WebSite Deployment Project

## 📋 Project Overview
This project demonstrates a complete Kubernetes deployment of a web application using **Kind (Kubernetes in Docker)** for local development. It includes cluster configuration, application deployment, and service exposure with port forwarding for external access.

## 🏗️ Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                     LOCAL MACHINE                               │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              KIND CLUSTER (4 NODES)                      │  │
│  │                                                          │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │  │
│  │  │  Ctrl    │  │ Worker   │  │ Worker   │  │ Worker   │  │  │
│  │  │  Plane   │  │   #1     │  │   #2     │  │   #3     │  │  │
│  │  │ [v1.33]  │  │          │  │          │  │          │  │  │
│  │  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘  │  │
│  │       │              │              │              │       │  │
│  │  ┌────▼──────────────▼──────────────▼──────────────▼────┐  │  │
│  │  │              Kubernetes Network                     │  │  │
│  │  └────┬──────────────────────────────┬─────────────────┘  │  │
│  │       │                              │                    │  │
│  │  ┌────▼────┐                  ┌──────▼──────┐            │  │
│  │  │  Pod    │                  │  Service    │            │  │
│  │  │gamer-web│                  │web-service  │            │  │
│  │  │replica1 │   Load           │   :80       │            │  │
│  │  └────┬────┘   Balancing      └──────┬──────┘            │  │
│  │       │        ┌────▼────┐           │                    │  │
│  │       └───────▶│  Pod    │◀──────────┘                    │  │
│  │                │gamer-web│                                 │  │
│  │                │replica2 │                                 │  │
│  │                └─────────┘                                 │  │
│  └─────────────────────────│──────────────────────────────────┘  │
│                            │                                      │
│                    ┌───────▼───────┐                             │
│                    │ Port Forward   │                             │
│                    │ 8000:80        │                             │
│                    └───────┬───────┘                             │
│                            │                                      │
│                    ┌───────▼───────┐                             │
│                    │   Accessible  │                             │
│                    │   at:8000     │                             │
│                    └───────────────┘                             │
│                                                                   │
└───────────────────────────────────────────────────────────────────┘
```

## 📁 Project Structure
```
kuber/
├── config.yml          # Kind cluster configuration
├── deploy-web.yml      # Web application deployment manifest
├── service-web.yml     # Service configuration
└── README.md          # This documentation file
```

## 🚀 Quick Start Guide

### Prerequisites
- **Docker** installed
- **Kind** (Kubernetes in Docker) installed
- **kubectl** configured

### Installation

#### 1. Install Kind
```bash
# For Linux/macOS
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.25.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# Verify installation
kind version
```

#### 2. Install kubectl
```bash
# For Linux
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Verify installation
kubectl version --client
```

### Step-by-Step Deployment

#### Step 1: Create the Kind Cluster
```bash
# Create cluster using configuration
kind create cluster --name k8s-web-cluster --config config.yml

# Verify cluster creation
kubectl get nodes
```

#### Step 2: Create Namespace
```bash
kubectl create namespace web-ns
kubectl get namespaces
```

#### Step 3: Deploy the Application
```bash
# Apply deployment manifest
kubectl apply -f deploy-web.yml

# Apply service manifest
kubectl apply -f service-web.yml

# Verify deployment
kubectl get all -n web-ns
```
#### Step 4: Access the Application
```bash
# Start port forwarding
kubectl port-forward service/web-service -n web-ns 8000:80 --address=0.0.0.0
```

**Access Options:**
- **Local machine**: Open browser and go to `http://localhost:8000`
- **From network**: Use machine IP `http://<YOUR_IP>:8000`
- **For EC2/Cloud instances**: Use public IP `http://<PUBLIC_IP>:8000`

## 📋 Configuration Files Details

### 1. **config.yml** - Kind Cluster Configuration
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    image: kindest/node:v1.33.1
  - role: worker
    image: kindest/node:v1.33.1
  - role: worker
    image: kindest/node:v1.33.1
  - role: worker
    image: kindest/node:v1.33.1
```

### 2. **deploy-web.yml** - Deployment Manifest
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
  namespace: web-ns
spec:
  replicas: 2
  selector:
    matchLabels:
      app: gamer-web
  template:
    metadata:
      labels:
        app: gamer-web
    spec:
      containers:
        - name: web-contnr
          image: manav1122/gamer-web:latest
```

### 3. **service-web.yml** - Service Manifest
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
  namespace: web-ns
spec:
  selector:
    app: gamer-web
  ports:
    - protocol: TCP
      port: 80       # Service port
      targetPort: 80 # Container port
```

## 🔧 Management Commands

### Monitoring
```bash
# Watch pods in real-time
kubectl get pods -n web-ns -w

# View detailed pod information
kubectl describe pod <pod-name> -n web-ns

# Check pod logs
kubectl logs <pod-name> -n web-ns
kubectl logs -f <pod-name> -n web-ns  # Follow logs

# Get service details
kubectl describe service web-service -n web-ns
```

### Scaling
```bash
# Scale up to 4 replicas
kubectl scale deployment web-deployment --replicas=4 -n web-ns

# Scale down to 1 replica
kubectl scale deployment web-deployment --replicas=1 -n web-ns

# Auto-scale based on CPU (if metrics server installed)
kubectl autoscale deployment web-deployment --cpu-percent=50 --min=2 --max=5 -n web-ns
```

### Updates and Rollbacks
```bash
# Update container image
kubectl set image deployment/web-deployment web-contnr=manav1122/gamer-web:v2 -n web-ns

# Check rollout status
kubectl rollout status deployment/web-deployment -n web-ns

# View rollout history
kubectl rollout history deployment/web-deployment -n web-ns

# Rollback to previous version
kubectl rollout undo deployment/web-deployment -n web-ns
```

## 🧹 Cleanup Commands

### Delete Specific Resources
```bash
# Delete deployment
kubectl delete -f deploy-web.yml

# Delete service
kubectl delete -f service-web.yml

# Delete namespace
kubectl delete namespace web-ns
```

### Complete Cleanup
```bash
# Delete all resources in namespace
kubectl delete all --all -n web-ns

# Delete the Kind cluster
kind delete cluster --name k8s-web-cluster

# List available clusters
kind get clusters
```

## 📊 Resource Verification Checklist

After deployment, verify everything is working:

1. ✅ **Cluster is running**: `kubectl get nodes`
2. ✅ **Namespace exists**: `kubectl get ns | grep web-ns`
3. ✅ **Pods are running**: `kubectl get pods -n web-ns`
4. ✅ **Service is created**: `kubectl get svc -n web-ns`
5. ✅ **Deployment is ready**: `kubectl get deployment -n web-ns`
6. ✅ **Port forwarding works**: `curl http://localhost:8000`
7. ✅ **Application responds**: Check browser at `http://localhost:8000`


## 🤝 Contributing
1. Fork the repository
2. Create a feature branch: `git checkout -b feature-name`
3. Commit changes: `git commit -m 'Add some feature'`
4. Push to branch: `git push origin feature-name`
5. Open a Pull Request

## 📄 License
This project is open-source and available under the MIT License.

## 🙏 Acknowledgments
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Kind Project](https://kind.sigs.k8s.io/)
- [Docker Hub](https://hub.docker.com/)
---

**Happy Kubernetting! 🎮**
