# GitOps Deployment with Argo CD on Azure Kubernetes Service (AKS)

This repository is part of a **Multi-Repo GitOps Architecture** that automates the deployment of the **Online Boutique microservices demo** into **Azure Kubernetes Service (AKS)** using **Argo CD**.

## Part 1: Multi-Repo GitOps Setup with Argo CD

###  GitOps Architecture Overview

This setup follows a **Multi-Repo GitOps model** based on Argo CD, which promotes separation of concerns across three repositories:

| Repository Name               | Purpose                                              | GitHub Link                                                                 |
|------------------------------|------------------------------------------------------|------------------------------------------------------------------------------|
| `online-shop-gitops-aks`     | Argo CD GitOps configuration (this repo)             | [online-shop-gitops-aks](https://github.com/DimitryZH/online-shop-gitops-aks) |
| `azure-terraform-online-shop`| Terraform-based provisioning of Azure infrastructure | [azure-terraform--online-shop](https://github.com/DimitryZH/azure-terraform-online-shop) |
| `microservices-demo`         | Microservices application code                       | [microservices-demo](https://github.com/DimitryZH/microservices-demo)       |


###  Key Benefits

- **Separation of Duties:** Code, infra, and deployment are decoupled.
- **Secure & Auditable:** Sensitive values handled via secure channels.
- **Declarative & Reproducible:** Source-controlled definitions for infra and workloads.
- **Scalable & Maintainable:** New environments/apps are easily added with the App of Apps pattern.


###  Architecture Diagram

The following diagram represents the interaction between these components:

![Multi-Repo GitOps Setup Diagram](./assets/gitops-architecture-diagram.png)

- **Argo CD** is installed inside the AKS cluster.
- It continuously monitors this GitOps repo for application deployment definitions.
- Application code is pulled from a separate repo into the cluster via Kubernetes manifests.
- Container images are stored in Docker Hub, built separately.
- Infrastructure like AKS is provisioned via Terraform.

## Cloud Migration & Cost Effectiveness

This project simulates a cloud migration scenario: moving the Online Boutique demo from Google Kubernetes Engine (GKE) to Azure Kubernetes Service (AKS).

The migration is driven by real-world DevOps and business factors:

- **Platform Alignment**: Many organizations prefer consolidating infrastructure in Azure for closer integration with Azure DevOps, Active Directory, and Microsoft 365 services.

- **Cost Optimization**:  
  AKS and GKE are both fully managed Kubernetes services, but they differ in pricing models — which can impact overall TCO (Total Cost of Ownership):

  **Control Plane Costs:**
  - **AKS**: Offers a free control plane — no additional hourly charge.
  - **GKE**: Charges $0.10/hour per cluster for the control plane (Standard mode).

  **GKE Autopilot Mode:**
  - Autopilot introduces a different pricing model, where charges are based on vCPU and memory usage for pods — in addition to the control plane fee.
  - While this offers granularity and optimization for certain workloads, it may lead to higher costs for steady or predictable traffic.

  **Worker Nodes:**
  - Both AKS and GKE charge for the underlying VM instances that power worker nodes.

  **Other Services:**
  - Both providers charge separately for storage, load balancers, networking, and ingress controllers.

  **Summary:**
  - **AKS is generally more cost-effective** for many use cases, particularly smaller clusters or those not needing specialized GKE features.
  - **GKE** offers more flexibility (e.g., Autopilot) that might be beneficial in dynamic or unpredictable environments, but comes with a premium.

  > In this context, migrating the Online Boutique app to AKS helps reduce infrastructure costs while maintaining scalability, security, and full GitOps automation.

- **Ecosystem Integration**: Azure-native tools like Azure Monitor, Key Vault, and Container Registry enhance visibility, security, and deployment speed.

- **Operational Consistency**: Terraform and Argo CD provide reproducible, declarative infrastructure and application management — enabling cross-cloud consistency and portability.





## Part 2: `online-shop-gitops-aks/` – Argo CD GitOps Repository

This repository holds Argo CD configurations using the **App of Apps** pattern to deploy the Online Boutique app to AKS.

###  Repository Structure

```bash
online-shop-gitops-aks/
├── apps/
│   ├── boutique-kustomize.yaml        # Argo CD Application for microservices demo
│   └── kustomization.yaml             # Kustomize entrypoint
├── projects/
│   └── online-boutique-project.yaml   # Argo CD App of Apps root project
└── assets/
    ├── gitops-architecture-diagram.png
    └── argocd-ui-screenshot.png       
```
 ###  Argo CD Installation

```bash
 kubectl create namespace argocd
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Access Argo CD UI
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
Default credentials:
```bash
username: admin
password: $(kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d)
```
###  Deploy App of Apps (bootstrap Argo CD)
```bash
kubectl apply -f projects/online-boutique-project.yaml -n argocd
```
This triggers deployment of all defined child applications under apps/.

### Argo CD UI Preview
Below is a snapshot of Argo CD’s web UI showing the deployed applications.


The `online-boutique-project` contains the `boutique-kustomize` child application.

Sync status and health are automatically displayed.

Each change in this repo gets reflected in AKS through continuous sync.

![Multi-Repo GitOps Setup Diagram](./assets/argocd-ui-screenshot.png)

### Results
Once all resources are provisioned via Terraform and apps are synced via Argo CD:

- Microservices are deployed to AKS.

- Argo CD provides continuous delivery with full visibility.

### Conclusions

This GitOps setup is:

- Modular: Clean separation of infrastructure, code, and deployment logic.

- Production-ready: Supports secure secrets management and GitOps workflows.

- Scalable: Easily extendable for more apps, environments, or clusters.

For future enhancements, consider integrating:

- Azure Key Vault for secure secret injection

- Automated CI pipelines for building and pushing images to ACR