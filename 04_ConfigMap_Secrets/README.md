# MongoDB + Mongo Express with ConfigMaps & Secrets (Kubernetes)

This example demonstrates how to deploy a MongoDB database and its web-based administration tool — Mongo Express — on Kubernetes using best practices for configuration management.

## 🎯 What You'll Learn

- ✅ **ConfigMaps** for environment configuration
- 🔐 **Secrets** for secure credential management
- 📦 **Services** for internal/external connectivity
- 🔁 **Deployments** for scalable pod management
- 🌐 **LoadBalancer** for external access
- 🐛 **Troubleshooting** real-world deployment issues

---

## 📁 Files Included

| File Name | Purpose |
|-----------|---------|
| `mongo.yml` | MongoDB deployment with persistent storage |
| `mongo-express.yml` | Mongo Express UI deployment |
| `mongo-secret.yaml` | Secure credentials (username/password) |
| `mongo-configmap.yml` | Non-sensitive configuration (optional) |
| `README.md` | This project overview |
| `troubleshooting.md` | Detailed debugging journey |

---

## 🚀 Quick Start

### Prerequisites

- Kubernetes cluster (GKE, Minikube, EKS, etc.)
- `kubectl` CLI configured
- (For GKE) `gcloud` CLI for firewall rules

### Deploy in Order

1. **Apply secrets first** (dependencies):
   ```bash
   kubectl apply -f mongo-secret.yaml
   kubectl apply -f mongo-configmap.yaml
   ```

2. **Deploy MongoDB**:
   ```bash
   kubectl apply -f mongo.yaml
   ```

3. **Deploy Mongo Express**:
   ```bash
   kubectl apply -f mongo-express.yml
   ```

4. **Verify deployment**:
   ```bash
   kubectl get pods
   kubectl get svc
   ```

5. **Access Mongo Express UI**:
   ```bash
   # Get external IP
   kubectl get svc mongo-express
   
   # Access via browser
   http://<EXTERNAL_IP>:8081
   ```

### One-Command Deployment

```bash
kubectl apply -f .
```

---

## 🔐 Default Credentials

| Field | Value |
|-------|-------|
| **Username** | `admin` |
| **Password** | `admin123` |

> 🔒 **Security Note**: Change these credentials in production by updating the `mongodb-secret.yaml` file with base64-encoded values.

---

## 🏗️ Architecture Overview

```
┌─────────────────────┐       ┌───────────────────────────┐
│  MongoDB Deployment │       │  Mongo Express Deployment │
│  (Pod x1)           │       │  (Pod x1)                 │
│  Port: 27017        │◄─────►│  Port: 8081               │
│  Env: Secrets       │       │  Env: Secrets + Server    │
│  Service: ClusterIP │       │  Service: LoadBalancer    │
└─────────────────────┘       └───────────────────────────┘
         ▲                             ▲
         │                             │
    ┌─────────────┐               ┌─────────────┐
    │   Secret    │               │ ConfigMap   │
    │ (mongodb-   │               │ (optional)  │
    │  secret)    │               │             │
    └─────────────┘               └─────────────┘
```

---

## 🌐 External Access Setup

### For Google Kubernetes Engine (GKE)

```bash
gcloud compute firewall-rules create allow-mongo-express \
  --allow tcp:8081 \
  --target-tags=gke-<cluster-name>-<node-pool-suffix> \
  --direction=INGRESS \
  --priority=1000 \
  --network=default
```

### For Minikube

```bash
minikube service mongo-express
```

### For Other Cloud Providers

Ensure your LoadBalancer service can create external IPs and configure security groups/firewall rules to allow traffic on port 8081.

---

## 🔍 Verification Commands

### Check Pod Status
```bash
kubectl get pods -o wide
kubectl describe pod <pod-name>
```

### Monitor Logs
```bash
kubectl logs -l app=mongo-express -f
kubectl logs -l app=mongodb -f
```

### Test Connectivity
```bash
# From within mongo-express pod
kubectl exec -it <mongo-express-pod> -- ping mongodb-service
kubectl exec -it <mongo-express-pod> -- telnet mongodb-service 27017
```

### Verify Configuration
```bash
kubectl get secret mongo-secret -o yaml
kubectl describe configmap mongo-configmap
```

---

## 🧠 Key Learning Outcomes

### 1. **Service Discovery**
- Services communicate via internal DNS names
- `mongodb-service` resolves to MongoDB pods
- Environment variable `ME_CONFIG_MONGODB_SERVER` must match service name

### 2. **Secret Management**
- Secrets are base64-encoded but decoded when injected into pods
- Apply secrets before deployments that depend on them
- Use `kubectl describe secret` to verify structure

### 3. **Networking**
- ClusterIP for internal services (MongoDB)
- LoadBalancer for external access (Mongo Express)
- Firewall rules required for cloud platforms

### 4. **Troubleshooting**
- `kubectl logs` for application errors
- `kubectl exec` for container inspection
- `kubectl describe` for resource details
- DNS resolution testing with `nslookup`/`ping`

---

## 🚨 Common Issues & Quick Fixes

| Issue | Quick Fix |
|-------|-----------|
| Connection refused | Check `ME_CONFIG_MONGODB_SERVER` matches service name |
| UI not accessible | Verify LoadBalancer IP and firewall rules |
| Authentication failed | Verify secret keys and base64 encoding |
| Pod crash loop | Check logs and ensure secrets exist |

📋 **For detailed troubleshooting**, see [troubleshooting.md](./troubleshooting.md)

---

## 🧪 Testing Your Deployment

### Smoke Test
```bash
# 1. Check all pods are running
kubectl get pods | grep -E "(mongo|express)"

# 2. Test web interface
curl -I http://<external-ip>:8081

# 3. Verify MongoDB connection
kubectl logs -l app=mongo-express | grep -i connected
```

### Integration Test
```bash
# Test authentication
kubectl exec -it <mongo-express-pod> -- \
  curl -u admin:admin123 http://localhost:8081/db/admin
```

---

## 🔄 Cleanup

```bash
# Remove all resources
kubectl delete -f .

# Remove firewall rule (GKE)
gcloud compute firewall-rules delete allow-mongo-express

# Verify cleanup
kubectl get all | grep -E "(mongo|express)"
```

---

## 📚 Additional Resources

- [Official MongoDB Kubernetes Documentation](https://docs.mongodb.com/kubernetes-operator/)
- [Mongo Express Docker Hub](https://hub.docker.com/_/mongo-express)
- [Kubernetes Secrets Best Practices](https://kubernetes.io/docs/concepts/configuration/secret/)
- [Troubleshooting Guide](./troubleshooting.md)

---

## 📝 Author & Credits

👤 **Rohini Swathi Bhatlapenumarthi**  
📅 **July 2025**  
🔗 **[LinkedIn Profile](https://www.linkedin.com/in/rohini-swathi-b-6a3029119)**  
📁 **[Full Troubleshooting Log](./troubleshooting.md)**

---

**Happy Kubernetes learning! 🚀**
