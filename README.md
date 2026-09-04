🚀 Kubernetes GitOps Local Cluster Platform

A production-grade, local GitOps-driven Kubernetes cluster built on kind with automated deployments via ArgoCD, automated SSL/TLS certificate management, custom NetworkPolicies, full-stack observability, and centralized dashboard management.

✨ Features & Components

📦 Local Multi-Node Cluster (kind): Isolated control plane and worker node architecture mimicking production setups.

🔄 GitOps Continuous Delivery (ArgoCD): Automated target branch tracking, continuous drift detection, prune, and selfHeal enforcement.

🛡️ Network Isolation (NetworkPolicy): Restricts ingress on port 5678 so that backend pods only accept incoming traffic from tier: frontend pods and the ingress-nginx ingress controller.

🔐 Automated TLS Termination (cert-manager): ClusterIssuer issuing SSL certificates dynamically for local hosts (myapp.local, api.local).

📊 Full-Stack Observability: Native containerized Prometheus metrics collection paired with dynamic Grafana visualization dashboards.

🔑 RBAC & Dashboard Access: Kubernetes Dashboard configured with a dedicated admin-user ServiceAccount and cluster-admin bindings.

📂 Repository Layout
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

🌐 Local Access Endpoint Directory
Service,Protocol / Access,URL / Local Endpoint,Default Namespace
🖥️ Frontend Application,🔐 HTTPS (SSL),https://myapp.local:8443/,🏷️ default
⚙️ Backend API,🔐 HTTPS (SSL),https://api.local:8443/,🏷️ default
🔄 ArgoCD Dashboard,🔐 HTTPS,https://localhost:9090/,🏷️ argocd
📊 Grafana,🌐 HTTP,http://localhost:3000/,🏷️ monitoring
📈 Prometheus,🌐 HTTP,http://localhost:9091/,🏷️ monitoring
☸️ Kubernetes Dashboard,🔀 HTTP Proxy,http://localhost:8001/api/v1/.../proxy/,🏷️ kubernetes-dashboard

🛠️ Quickstart & Deployment Instructions
1️⃣ Cluster Initialization
Create the kind cluster using the local configuration file:

Bash
kind create cluster --config kind-config.yaml --name kind
2️⃣ Deploy Infrastructure & Ingress Controllers
Apply NGINX Ingress Controller and cert-manager:

Bash
# Install NGINX Ingress
kubectl apply -f [https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml](https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml)

# Install cert-manager
kubectl apply -f [https://github.com/cert-manager/cert-manager/releases/download/v1.14.4/cert-manager.yaml](https://github.com/cert-manager/cert-manager/releases/download/v1.14.4/cert-manager.yaml)

# Apply ClusterIssuer
kubectl apply -f cluster-issuer.yaml
3️⃣ Setup Local Domain Host Entries
Add local domain mappings to /etc/hosts:

Bash
echo "127.0.0.1 myapp.local api.local" | sudo tee -a /etc/hosts
4️⃣ Deploy GitOps Automation (ArgoCD)
Install ArgoCD and register the application pipeline:

Bash
kubectl create namespace argocd
kubectl apply -n argocd -f [https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml](https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml)

# Apply the GitOps Application definition
kubectl apply -f gitops-repo/argocd-app.yaml
🧪 Verification & Security Auditing
🔒 Verify Backend Network Policy Isolation
Bash
# 1. Test allowed connection from frontend pod (Should succeed)
FRONTEND_POD=$(kubectl get pod -l tier=frontend -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $FRONTEND_POD -- wget -qO- http://backend-api:5678/

# 2. Test blocked connection from an unauthorized pod (Should time out)
kubectl run test-unauthorized --image=busybox --restart=Never -- timeout 5 wget -qO- http://backend-api:5678/
🔄 Check ArgoCD Synchronization Status
Bash
kubectl get application fullstack-k8s-app -n argocd


📜 License
Distributed under the MIT License. See LICENSE for details.
"""

with open("README.md", "w") as f:
f.write(readme_content)

print("Updated README.md with icons across all sections.")


```text?code_stdout&code_event_index=1
Updated README.md with icons across all sections.

The README.md file has been updated with relevant icons across every section, bullet point, table row, file directory entry, and architecture component.

# 🚀 Kubernetes GitOps Local Cluster Platform

[![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.30-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![kind](https://img.shields.io/badge/kind-v0.22-2088FF?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kind.sigs.k8s.io/)
[![ArgoCD](https://img.shields.io/badge/Argo%20CD-v2.10-EF7B4D?style=for-the-badge&logo=argo&logoColor=white)](https://argoproj.github.io/cd/)
[![NGINX Ingress](https://img.shields.io/badge/NGINX_Ingress-v1.10-009639?style=for-the-badge&logo=nginx&logoColor=white)](https://kubernetes.github.io/ingress-nginx/)
[![cert-manager](https://img.shields.io/badge/cert--manager-v1.14-326CE5?style=for-the-badge&logo=certmanager&logoColor=white)](https://cert-manager.io/)
[![Prometheus](https://img.shields.io/badge/Prometheus-v2.50-E6522C?style=for-the-badge&logo=prometheus&logoColor=white)](https://prometheus.io/)
[![Grafana](https://img.shields.io/badge/Grafana-v10.3-F46800?style=for-the-badge&logo=grafana&logoColor=white)](https://grafana.com/)

A production-grade, local GitOps-driven Kubernetes cluster built on **kind** with automated deployments via **ArgoCD**, automated **SSL/TLS certificate management**, custom **NetworkPolicies**, full-stack **observability**, and centralized dashboard management.

---

## 📐 Architecture Overview

```text
                        ┌─────────────────────────────────────────┐
                        │              💻 Host Machine            │
                        └────────────────────┬────────────────────┘
                                             │
                       ┌─────────────────────┴─────────────────────┐
                       │           🐳 kind Kubernetes Cluster      │
                       │                                           │
                       │  ┌─────────────────────────────────────┐  │
                       │  │         🌐 ingress-nginx (8443)     │  │
                       │  └──────────────────┬──────────────────┘  │
                       │                     │ 🔐 (TLS Termination)│
                       │          ┌──────────┴──────────┐          │
                       │          ▼                     ▼          │
                       │   ┌─────────────┐       ┌─────────────┐   │
                       │   │ myapp.local │       │  api.local  │   │
                       │   └──────┬──────┘       └──────┬──────┘   │
                       │          │                     │          │
                       │          ▼                     ▼          │
                       │   ┌─────────────┐  🔌 port 5678┌─────────┐│
                       │   │ 🖥️ Frontend │─────────────►│⚙️Backend││
                       │   │ Pods (x1)   │🛡️NetPolicy   │ Pods(x3)││
                       │   └─────────────┘              └─────────┘│
                       └───────────────────────────────────────────┘
```

---

## ✨ Features & Components

* 📦 **Local Multi-Node Cluster (`kind`):** Isolated control plane and worker node architecture mimicking production setups.
* 🔄 **GitOps Continuous Delivery (`ArgoCD`):** Automated target branch tracking, continuous drift detection, `prune`, and `selfHeal` enforcement.
* 🛡️ **Network Isolation (`NetworkPolicy`):** Restricts ingress on port `5678` so that backend pods only accept incoming traffic from `tier: frontend` pods and the `ingress-nginx` ingress controller.
* 🔐 **Automated TLS Termination (`cert-manager`):** `ClusterIssuer` issuing SSL certificates dynamically for local hosts (`myapp.local`, `api.local`).
* 📊 **Full-Stack Observability:** Native containerized **Prometheus** metrics collection paired with dynamic **Grafana** visualization dashboards.
* 🔑 **RBAC & Dashboard Access:** Kubernetes Dashboard configured with a dedicated `admin-user` ServiceAccount and `cluster-admin` bindings.

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

### 🔄 Check ArgoCD Synchronization Status
```bash
kubectl get application fullstack-k8s-app -n argocd
```

---

## 📜 License
Distributed under the MIT License. See `LICENSE` for details.
