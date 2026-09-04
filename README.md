# 🚀 Kubernetes GitOps Local Cluster Platform

[![Docker](https://img.shields.io/badge/Docker-v26.0-2496ED?style=flat-square&logo=docker&logoColor=white)](https://www.docker.com/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.30-326CE5?style=flat-square&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![kind](https://img.shields.io/badge/kind-v0.22-46A2F1?style=flat-square&logo=kubernetes&logoColor=white)](https://kind.sigs.k8s.io/)
[![cert-manager](https://img.shields.io/badge/cert--manager-v1.14-00C0A3?style=flat-square&logo=certmanager&logoColor=white)](https://cert-manager.io/)
[![NGINX Ingress](https://img.shields.io/badge/NGINX_Ingress-v1.10-009639?style=flat-square&logo=nginx&logoColor=white)](https://kubernetes.github.io/ingress-nginx/)
[![Grafana](https://img.shields.io/badge/Grafana-v10.3-F46800?style=flat-square&logo=grafana&logoColor=white)](https://grafana.com/)
[![Prometheus](https://img.shields.io/badge/Prometheus-v2.50-E6522C?style=flat-square&logo=prometheus&logoColor=white)](https://prometheus.io/)
[![ArgoCD](https://img.shields.io/badge/ArgoCD-v2.10-EF7B4D?style=flat-square&logo=argo&logoColor=white)](https://argoproj.github.io/cd/)

A production-grade, local GitOps-driven Kubernetes cluster platform built on Docker Engine and kind featuring automated deployments via ArgoCD, dynamic SSL/TLS certificate management, zero-trust NetworkPolicies, full-stack containerized observability, and centralized dashboard controls.

---

## 📐 Architecture Overview

```text
                        ┌─────────────────────────────────────────┐
                        │               Host Machine              │
                        │             (Docker Engine)             │
                        └────────────────────┬────────────────────┘
                                             │
                       ┌─────────────────────┴─────────────────────┐
                       │       kind Cluster (Docker Nodes)         │
                       │                                           │
                       │  ┌─────────────────────────────────────┐  │
                       │  │         ingress-nginx (8443)        │  │
                       │  └──────────────────┬──────────────────┘  │
                       │                     │ (TLS Termination)   │
                       │          ┌──────────┴──────────┐          │
                       │          ▼                     ▼          │
                       │   ┌─────────────┐       ┌─────────────┐   │
                       │   │ myapp.local │       │  api.local  │   │
                       │   └──────┬──────┘       └──────┬──────┘   │
                       │          │                     │          │
                       │          ▼                     ▼          │
                       │   ┌─────────────┐  port 5678   ┌─────────┐│
                       │   │  Frontend   ├─────────────►│ Backend ││
                       │   │  Pods (x1)  │ NetworkPolicy│ Pods(x3)││
                       │   └─────────────┘              └─────────┘│
                       │                                           │
                       │  ┌─────────────────────────────────────┐  │
                       │  │      Docker Observability Stack     │  │
                       │  │   (Prometheus & Grafana Containers) │  │
                       │  └─────────────────────────────────────┘  │
                       └───────────────────────────────────────────┘

```

---

## ✨ Features & Components
* 🔑 **RBAC & Dashboard Access:** Kubernetes Dashboard configured with a dedicated `admin-user` ServiceAccount and `cluster-admin` bindings.
* 📦 **Local Multi-Node Cluster (`kind`):** Isolated control plane and worker node architecture mimicking production setups.
* 🐳 Docker Engine Container Runtime: Serves as the primary virtualization layer, managing node containers for kind and isolated observability runtime environments.
* 🔄 **GitOps Continuous Delivery (`ArgoCD`):** Automated target branch tracking, continuous drift detection, `prune`, and `selfHeal` enforcement.
* 🛡️ **Network Isolation (`NetworkPolicy`):** Restricts ingress on port `5678` so that backend pods only accept incoming traffic from `tier: frontend` pods and the `ingress-nginx` ingress controller.
* 🔐 **Automated TLS Termination (`cert-manager`):** `ClusterIssuer` issuing SSL certificates dynamically for local hosts (`myapp.local`, `api.local`).
* 📊 **Full-Stack Observability:** Native containerized **Prometheus** metrics collection paired with dynamic **Grafana** visualization dashboards.

---

## 📂 Repository Layout

```text
.
├── 📜 cluster-issuer.yaml             # cert-manager ClusterIssuer configuration
├── 🛡️ dashboard-admin.yaml            # ServiceAccount & ClusterRoleBinding for Dashboard
├── 🐳 kind-config.yaml                # Multi-node kind cluster definition
├── 📁 gitops-repo/
│   ├── 🔄 argocd-app.yaml             # ArgoCD Application resource
│   ├── 📂 base/
│   │   ├── ⚙️ apps-and-services.yaml  # Deployment & Service definitions (Frontend/Backend)
│   │   ├── 🔒 backend-network-policy.yaml # Restricted ingress NetworkPolicy (port 5678)
│   │   ├── 🌐 ingress-rules.yaml      # NGINX Ingress rules with TLS configuration
│   │   └── 🧩 kustomization.yaml      # Kustomize base manifest aggregator
│   ├── 📂 monitoring/
│   │   ├── 📈 prometheus.yaml         # Containerized Prometheus deployment
│   │   └── 📊 grafana.yaml            # Containerized Grafana deployment
│   └── 📂 overlays/
│       └── 🛠️ dev/                   # Environment-specific Kustomize overlays
└── 📄 README.md
```

---

## 🌐 Local Access Endpoint Directory

| Service | Protocol / Access | URL / Local Endpoint | Default Namespace |
| :--- | :--- | :--- | :--- |
| 🖥️ **Frontend Application** | 🔐 `HTTPS (SSL)` | `https://myapp.local:8443/` | 🏷️ `default` |
| ⚙️ **Backend API** | 🔐 `HTTPS (SSL)` | `https://api.local:8443/` | 🏷️ `default` |
| 🔄 **ArgoCD Dashboard** | 🔐 `HTTPS` | `https://localhost:9090/` | 🏷️ `argocd` |
| 📊 **Grafana** | 🌐 `HTTP` | `http://localhost:3000/` | 🏷️ `monitoring` |
| 📈 **Prometheus** | 🌐 `HTTP` | `http://localhost:9091/` | 🏷️ `monitoring` |
| ☸️ **Kubernetes Dashboard** | 🔀 `HTTP Proxy` | `http://localhost:8001/api/v1/.../proxy/` | 🏷️ `kubernetes-dashboard` |

---

## 🛠️ Quickstart & Deployment Instructions

### 1️⃣ Cluster Initialization
Create the `kind` cluster using the local configuration file:
```bash
kind create cluster --config kind-config.yaml --name kind
```

### 2️⃣ Deploy Infrastructure & Ingress Controllers
Apply NGINX Ingress Controller and cert-manager:
```bash
# Install NGINX Ingress
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml

# Install cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.4/cert-manager.yaml

# Apply ClusterIssuer
kubectl apply -f cluster-issuer.yaml
```

### 3️⃣ Setup Local Domain Host Entries
Add local domain mappings to `/etc/hosts`:
```bash
echo "127.0.0.1 myapp.local api.local" | sudo tee -a /etc/hosts
```

### 4️⃣ Deploy GitOps Automation (ArgoCD)
Install ArgoCD and register the application pipeline:
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Apply the GitOps Application definition
kubectl apply -f gitops-repo/argocd-app.yaml
```

---

## 🧪 Verification & Security Auditing

### 🔒 Verify Backend Network Policy Isolation
```bash
# 1. Test allowed connection from frontend pod (Should succeed)
FRONTEND_POD=$(kubectl get pod -l tier=frontend -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $FRONTEND_POD -- wget -qO- http://backend-api:5678/

# 2. Test blocked connection from an unauthorized pod (Should time out)
kubectl run test-unauthorized --image=busybox --restart=Never -- timeout 5 wget -qO- http://backend-api:5678/
```
### 🐳 Inspect Docker Node Containers
```Bash
# Verify active kind cluster node containers on Docker
docker ps --filter "label=io.x-k8s.kind.cluster=kind"
```
### 🔄 Check ArgoCD Synchronization Status
```bash
kubectl get application fullstack-k8s-app -n argocd
```

🛠️ Troubleshooting & Docker Diagnostics
🔍 Docker Health & Container Logs
```Bash
# View Docker container logs for the control-plane node
docker logs kind-control-plane --tail 100 -f

# Inspect the underlying Docker network bridging kind nodes
docker network inspect kind
```
🌐 Ingress & Certificate Inspection
```Bash
# Check Ingress Controller status
kubectl get pods -n ingress-nginx -l app.kubernetes.io/component=controller

# Check issued cert-manager certificates
kubectl get certificate -A
```

---


## 📜 License
Distributed under the MIT License. See `LICENSE` for details.
