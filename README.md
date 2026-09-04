# 🚀 Kubernetes GitOps Local Cluster Platform

[![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.30-181717?style=flat-square&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![kind](https://img.shields.io/badge/kind-v0.22-181717?style=flat-square&logo=kubernetes&logoColor=white)](https://kind.sigs.k8s.io/)
[![ArgoCD](https://img.shields.io/badge/ArgoCD-v2.10-181717?style=flat-square&logo=argo&logoColor=white)](https://argoproj.github.io/cd/)
[![NGINX Ingress](https://img.shields.io/badge/NGINX_Ingress-v1.10-181717?style=flat-square&logo=nginx&logoColor=white)](https://kubernetes.github.io/ingress-nginx/)
[![cert-manager](https://img.shields.io/badge/cert--manager-v1.14-181717?style=flat-square&logo=certmanager&logoColor=white)](https://cert-manager.io/)
[![Prometheus](https://img.shields.io/badge/Prometheus-v2.50-181717?style=flat-square&logo=prometheus&logoColor=white)](https://prometheus.io/)
[![Grafana](https://img.shields.io/badge/Grafana-v10.3-181717?style=flat-square&logo=grafana&logoColor=white)](https://grafana.com/)

A production-grade, local GitOps-driven Kubernetes cluster built on **kind** with automated deployments via **ArgoCD**, automated **SSL/TLS certificate management**, custom **NetworkPolicies**, full-stack **observability**, and centralized dashboard management.

---

## 📐 Architecture Overview

```text
                        ┌─────────────────────────────────────────┐
                        │               Host Machine              │
                        └────────────────────┬────────────────────┘
                                             │
                       ┌─────────────────────┴─────────────────────┐
                       │           kind Kubernetes Cluster         │
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
                       └───────────────────────────────────────────┘
```

---

## 🛠️ Key Capabilities

| Category | Component | Technical Specification |
| :--- | :--- | :--- |
| **Cluster Engine** | Local Multi-Node (`kind`) | Mimics multi-node production setup (1 control plane, 3 workers). |
| **GitOps Engine** | ArgoCD | Automated branch tracking, drift detection, pruning, and self-healing. |
| **Network Security** | NetworkPolicy | Restricts port 5678 access exclusively to `tier: frontend` and Ingress. |
| **Certificates** | cert-manager | Dynamic local SSL/TLS certificate issuance via `ClusterIssuer`. |
| **Observability** | Prometheus & Grafana | Native metric collection with dynamic visualization dashboards. |
| **Access Control** | K8s Dashboard | ServiceAccount-based authentication bound to `cluster-admin`. |

---

## 📂 Repository Structure

| File / Path | Type | Functional Description |
| :--- | :--- | :--- |
| `cluster-issuer.yaml` | Manifest | Configures cert-manager ClusterIssuer for local TLS. |
| `dashboard-admin.yaml` | Manifest | Admin ServiceAccount and ClusterRoleBinding for Dashboard. |
| `kind-config.yaml` | Config | Definition file for the multi-node kind cluster layout. |
| `gitops-repo/` | Directory | Core repository folder synchronized by ArgoCD. |
| `gitops-repo/argocd-app.yaml` | Manifest | Declarative ArgoCD Application targeting the `main` branch. |
| `gitops-repo/base/` | Directory | Kustomize base folder containing core cluster manifests. |
| `gitops-repo/base/apps-and-services.yaml` | Manifest | Deployments and Services for frontend and backend applications. |
| `gitops-repo/base/backend-network-policy.yaml` | Manifest | Restricts ingress traffic on backend port 5678. |
| `gitops-repo/base/ingress-rules.yaml` | Manifest | Defines NGINX Ingress routes with host-based TLS termination. |
| `gitops-repo/base/kustomization.yaml` | Kustomize | Aggregates all base Kubernetes resources. |
| `gitops-repo/monitoring/` | Directory | Deployment manifests for Prometheus and Grafana. |
| `gitops-repo/overlays/dev/` | Directory | Kustomize patch overlays for development settings. |

---

## 🌐 Local Access Endpoints

| Service | Protocol | Access Endpoint / URL | Namespace |
| :--- | :--- | :--- | :--- |
| **Frontend Application** | HTTPS (SSL) | `https://myapp.local:8443/` | `default` |
| **Backend API** | HTTPS (SSL) | `https://api.local:8443/` | `default` |
| **ArgoCD Dashboard** | HTTPS | `https://localhost:9090/` | `argocd` |
| **Grafana** | HTTP | `http://localhost:3000/` | `monitoring` |
| **Prometheus** | HTTP | `http://localhost:9091/` | `monitoring` |
| **Kubernetes Dashboard** | HTTP Proxy | `http://localhost:8001/api/v1/.../proxy/` | `kubernetes-dashboard` |

---

## 🚀 Deployment Instructions

| Stage | Step | Command |
| :--- | :--- | :--- |
| **1. Cluster** | Create kind cluster | `kind create cluster --config kind-config.yaml --name kind` |
| **2. Ingress & TLS** | Deploy Ingress & cert-manager | `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml`<br>`kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.4/cert-manager.yaml`<br>`kubectl apply -f cluster-issuer.yaml` |
| **3. DNS** | Configure local host routing | `echo "127.0.0.1 myapp.local api.local" \| sudo tee -a /etc/hosts` |
| **4. GitOps** | Install & Register ArgoCD | `kubectl create namespace argocd`<br>`kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml`<br>`kubectl apply -f gitops-repo/argocd-app.yaml` |

---

## 🧪 Security & Policy Verification

| Verification Target | Test Description | Command | Expected Result |
| :--- | :--- | :--- | :--- |
| **Allowed Access** | Frontend -> Backend | `FRONTEND_POD=$(kubectl get pod -l tier=frontend -o jsonpath='{.items[0].metadata.name}')`<br>`kubectl exec -it $FRONTEND_POD -- wget -qO- http://backend-api:5678/` | **HTTP 200 OK** |
| **Blocked Access** | Rogue Pod -> Backend | `kubectl run test-unauthorized --image=busybox --restart=Never -- timeout 5 wget -qO- http://backend-api:5678/` | **Timed out (Blocked)** |
| **GitOps Sync** | ArgoCD Pipeline | `kubectl get application fullstack-k8s-app -n argocd` | **Synced / Healthy** |

---

## 📜 License
Distributed under the MIT License. See `LICENSE` for details.
