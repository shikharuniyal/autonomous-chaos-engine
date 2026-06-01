# DARWIN Quick Start Guide

## Installation Steps (Run in Order)

### Step 1: Install Base Tools

Open **PowerShell as Administrator**:

```powershell
cd e:\#EditorCodes\banglore_Hackthon
.\setup\install_windows.ps1
```

**Note:** If Go or Node.js installers are downloaded, run them manually and restart PowerShell.

---

### Step 2: Verify Installation

```powershell
.\setup\verify_installation.ps1
```

**Expected:** All tools should show ✓ (green checkmark)

---

### Step 3: Start Kubernetes Cluster

```powershell
minikube start --cpus=4 --memory=8192 --disk-size=40gb --driver=docker
```

**Wait 3-5 minutes.** Then verify:

```powershell
kubectl get nodes
# Should show: minikube   Ready    control-plane
```

---

### Step 4: Enable Minikube Addons

```powershell
minikube addons enable metrics-server
minikube addons enable ingress
```

---

### Step 5: Install Istio Service Mesh

```powershell
.\setup\install_istio.ps1
```

**Verify:**
```powershell
kubectl get pods -n istio-system
# All pods should be Running
```

---

### Step 6: Install Databases

```powershell
.\setup\install_databases.ps1
```

**This takes 5-10 minutes.** Helm will install:
- Neo4j (Knowledge Graph)
- PostgreSQL (DNA Store)
- Redis (Cache)
- NATS (Message Bus)
- Prometheus + Grafana (Observability)
- Loki (Logs)
- Jaeger (Tracing)

**Verify:**
```powershell
kubectl get pods -n darwin
# All pods should eventually be Running (wait 5 min)
```

---

## Accessing Services

### Neo4j Browser (Knowledge Graph)

```powershell
kubectl port-forward svc/neo4j 7474:7474 -n darwin
```

Open: http://localhost:7474
- Username: `neo4j`
- Password: `darwin123`

---

### Grafana (Metrics Dashboard)

```powershell
kubectl port-forward svc/prometheus-grafana 3000:80 -n darwin
```

Open: http://localhost:3000
- Username: `admin`
- Password: `darwin123`

---

### Prometheus (Raw Metrics)

```powershell
kubectl port-forward svc/prometheus-operated 9090:9090 -n darwin
```

Open: http://localhost:9090

---

### Jaeger (Distributed Tracing)

```powershell
kubectl port-forward svc/jaeger-query 16686:16686 -n darwin
```

Open: http://localhost:16686

---

## Troubleshooting

### Minikube won't start

```powershell
minikube delete
minikube start --cpus=4 --memory=8192 --driver=docker
```

### Pods stuck in "Pending"

```powershell
# Check events
kubectl get events -n darwin --sort-by='.lastTimestamp'

# Check pod details
kubectl describe pod <pod-name> -n darwin
```

Common issues:
- **Insufficient memory:** Increase `--memory` to 10240
- **Image pull errors:** Check Docker is running

### Database won't install

```powershell
# Delete and reinstall
helm uninstall neo4j -n darwin
helm uninstall postgresql -n darwin
helm uninstall redis -n darwin

# Then run install script again
.\setup\install_databases.ps1
```

---

## Next Steps

After infrastructure is ready:

1. **Install Python dependencies:**
   ```powershell
   pip install kubernetes scikit-learn torch psycopg2-binary neo4j redis nats-py prometheus-client pyyaml joblib
   ```

2. **Seed Neo4j Knowledge Graph:**
   ```powershell
   # Port-forward Neo4j first
   kubectl port-forward svc/neo4j 7687:7687 -n darwin

   # In another terminal
   python scripts/seed_knowledge_graph.py
   ```

3. **Generate ML Training Data:**
   ```powershell
   python scripts/generate_training_data.py --runs 50
   ```

4. **Train ML Models:**
   ```powershell
   python scripts/train_classifier.py
   python scripts/train_lstm.py
   ```

5. **Build Microservices:**
   ```powershell
   cd services
   docker build -t darwin/auth-service:v1 ./auth-service
   docker build -t darwin/api-gateway:v1 ./api-gateway
   docker build -t darwin/payment-service:v1 ./payment-service
   docker build -t darwin/order-service:v1 ./order-service
   docker build -t darwin/inventory-service:v1 ./inventory-service
   docker build -t darwin/notification-service:v1 ./notification-service

   # Load into minikube
   minikube image load darwin/auth-service:v1
   minikube image load darwin/api-gateway:v1
   minikube image load darwin/payment-service:v1
   minikube image load darwin/order-service:v1
   minikube image load darwin/inventory-service:v1
   minikube image load darwin/notification-service:v1
   ```

6. **Deploy DARWIN:**
   ```powershell
   kubectl apply -f k8s/services/
   kubectl apply -f k8s/virus/
   kubectl apply -f k8s/antibody/
   kubectl apply -f k8s/detector/
   kubectl apply -f k8s/dashboard/
   ```

7. **Check deployment:**
   ```powershell
   kubectl get pods -n darwin
   ```

8. **Access dashboard:**
   ```powershell
   kubectl port-forward svc/darwin-dashboard 3001:3000 -n darwin
   ```

   Open: http://localhost:3001

---

## Useful Commands

```powershell
# View all resources
kubectl get all -n darwin

# View logs
kubectl logs <pod-name> -n darwin
kubectl logs <pod-name> -c istio-proxy -n darwin  # Istio sidecar logs

# Shell into pod
kubectl exec -it <pod-name> -n darwin -- /bin/bash

# Delete everything and start fresh
minikube delete
helm uninstall --namespace darwin neo4j postgresql redis nats prometheus loki jaeger
kubectl delete namespace darwin
kubectl delete namespace darwin-honeypot

# Then start from Step 3 above
```

---

## Resource Monitor

```powershell
# Watch resource usage
kubectl top nodes
kubectl top pods -n darwin

# Minikube dashboard (visual)
minikube dashboard
```

---

## Status Checklist

Before running demos, ensure:

- ✅ Minikube is running: `minikube status`
- ✅ All istio-system pods are Running: `kubectl get pods -n istio-system`
- ✅ All darwin pods are Running: `kubectl get pods -n darwin`
- ✅ Neo4j is seeded with knowledge graph
- ✅ ML models are trained (models/*.joblib, models/*.pt exist)
- ✅ Microservices are deployed
- ✅ Dashboard is accessible

---

**You're ready to demo DARWIN!** 🎉
