# 🚀 Kubernetes GitOps Local Cluster Platform

[![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.30-326CE5?style=flat-square&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Docker](https://img.shields.io/badge/Docker-v26.0-2496ED?style=flat-square&logo=docker&logoColor=white)](https://www.docker.com/)
[![kind](https://img.shields.io/badge/kind-v0.22-46A2F1?style=flat-square&logo=kubernetes&logoColor=white)](https://kind.sigs.k8s.io/)
[![ArgoCD](https://img.shields.io/badge/ArgoCD-v2.10-EF7B4D?style=flat-square&logo=argo&logoColor=white)](https://argoproj.github.io/cd/)
[![NGINX Ingress](https://img.shields.io/badge/NGINX_Ingress-v1.10-009639?style=flat-square&logo=nginx&logoColor=white)](https://kubernetes.github.io/ingress-nginx/)
[![cert-manager](https://img.shields.io/badge/cert--manager-v1.14-00C0A3?style=flat-square&logo=certmanager&logoColor=white)](https://cert-manager.io/)
[![Prometheus](https://img.shields.io/badge/Prometheus-v2.50-E6522C?style=flat-square&logo=prometheus&logoColor=white)](https://prometheus.io/)
[![Grafana](https://img.shields.io/badge/Grafana-v10.3-F46800?style=flat-square&logo=grafana&logoColor=white)](https://grafana.com/)

A production-grade, local GitOps-driven Kubernetes cluster platform built on **Docker** and **kind** featuring automated deployments via **ArgoCD**, dynamic **SSL/TLS certificate management**, zero-trust **NetworkPolicies**, full-stack **observability**, and centralized dashboard controls.

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


🛠️ Key CapabilitiesCategoryComponentTechnical Specification🐳 Container RuntimeDocker EngineCore virtualization runtime powering kind cluster nodes and observability services.☸️ Cluster EngineLocal Multi-Node (kind)Production-matched multi-node topology featuring 1 control plane and 3 worker nodes.🔄 GitOps EngineArgoCDDeclarative continuous delivery with automated drift detection, prune, and selfHeal.🛡️ Network SecurityNetworkPolicyZero-trust isolation restricting port 5678 exclusively to tier: frontend and Ingress.🔐 Certificate Managementcert-managerAutomated local SSL/TLS certificate generation and lifecycle control via ClusterIssuer.📊 Observability StackPrometheus & GrafanaNative metric collection, real-time alerting, and custom visual performance dashboards.🔑 Access ControlK8s DashboardRole-based access control (RBAC) managed via admin-user ServiceAccount.📂 Repository StructureFile / Directory PathItem TypeFunctional Description📜 cluster-issuer.yamlK8s ManifestConfigures local self-signed ClusterIssuer for cert-manager.🛡️ dashboard-admin.yamlRBAC ConfigDefines ServiceAccount and ClusterRoleBinding for K8s Dashboard.🐳 kind-config.yamlCluster SpecNode mapping and port-forwarding layout for the multi-node cluster.📁 gitops-repo/Root FolderPrimary Git repository path watched and synchronized by ArgoCD.🔄 gitops-repo/argocd-app.yamlApplicationDeclarative ArgoCD Application resource linking target Git repo.📂 gitops-repo/base/Kustomize BaseCore microservice application manifests and service definitions.⚙️ gitops-repo/base/apps-and-services.yamlWorkloadsDeployment and Service specs for frontend and backend workloads.🔒 gitops-repo/base/backend-network-policy.yamlSecurity PolicyIngress restriction policy isolating backend services.🌐 gitops-repo/base/ingress-rules.yamlRouting SpecHost-based Ingress paths with TLS certificate termination rules.🧩 gitops-repo/base/kustomization.yamlEntry PointAggregates all underlying Kustomize resource manifests.📈 gitops-repo/monitoring/Stack ConfigDeployment definitions for Prometheus scrapers and Grafana dashboards.🛠️ gitops-repo/overlays/dev/EnvironmentEnvironment-specific patch overrides tailored for dev workflows.🌐 Local Access EndpointsService TargetProtocol / TypeEndpoint / Access URLTarget Namespace💻 Frontend Web App🔐 HTTPS (SSL)https://myapp.local:8443/default⚙️ Backend REST API🔐 HTTPS (SSL)https://api.local:8443/default🔄 ArgoCD Console🔐 HTTPShttps://localhost:9090/argocd📊 Grafana Dashboards🌐 HTTPhttp://localhost:3000/monitoring📈 Prometheus Metrics🌐 HTTPhttp://localhost:9091/monitoring☸️ Kubernetes Dashboard🔀 HTTP Proxyhttp://localhost:8001/api/v1/.../proxy/kubernetes-dashboard🚀 Deployment InstructionsExecution StageObjectiveTarget Command1️⃣ Docker RuntimeStart Docker Enginesystemctl start docker (or open Docker Desktop)2️⃣ Cluster ProvisioningInitialize local kind clusterkind create cluster --config kind-config.yaml --name kind3️⃣ Platform ServicesDeploy NGINX & cert-managerkubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yamlkubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.4/cert-manager.yamlkubectl apply -f cluster-issuer.yaml4️⃣ DNS ResolutionRegister host recordsecho "127.0.0.1 myapp.local api.local" | sudo tee -a /etc/hosts5️⃣ GitOps & ObservabilityBootstrap ArgoCD & Monitoringkubectl create namespace argocdkubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yamlkubectl apply -f gitops-repo/argocd-app.yaml🧪 Security & Policy VerificationVerification TargetTest ObjectiveTest CommandExpected Result🟢 Allowed TrafficFrontend → BackendFRONTEND_POD=$(kubectl get pod -l tier=frontend -o jsonpath='{.items[0].metadata.name}')kubectl exec -it $FRONTEND_POD -- wget -qO- http://backend-api:5678/HTTP 200 OK🔴 Blocked TrafficRogue Pod → Backendkubectl run test-unauthorized --image=busybox --restart=Never -- timeout 5 wget -qO- http://backend-api:5678/Connection Timed Out🔄 GitOps PipelineSync State Verificationkubectl get application fullstack-k8s-app -n argocdSynced / Healthy📜 LicenseDistributed under the MIT License. See LICENSE for details.
