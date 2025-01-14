# ArgoCD App of Apps

This repository demonstrates the "App of Apps" pattern using ArgoCD. The "App of Apps" pattern allows you to manage multiple applications with a single ArgoCD application.

## Prerequisites

- Kubernetes cluster
- ArgoCD installed and configured
- kubectl configured to interact with your cluster

## Repository Structure

app-of-apps/
|_ .helmignore
|_ Chart.yaml
|_ templates/
| |_ _application.yaml
| |_ _chart.yaml
| |_ application.yaml
| |_ chart.yaml
|_ values.yaml
apps/
|_ argocd/
| |_ argocd.yaml
| |_ kustomization.yaml
|_ charts/
| |_ sealed-secrets/
| | |_ .helmignore
| | |_ Chart.lock
| | |_ Chart.yaml
| | |_ charts/
| | | |_ sealed-secrets-2.12.0.tgz
| | |_ values.yaml
|_ root-application.yaml

### Main Components

- **app-of-apps**: The main Helm chart that includes templates and values for deploying multiple applications.
- **apps**: Directory containing individual application configurations.
- **charts**: Directory containing sub-charts and dependencies.

## How It Works

### Helm Chart

The main Helm chart is defined in the `app-of-apps` directory. It includes the following key files:

- `Chart.yaml`: Defines the chart metadata, including dependencies.
- `values.yaml`: Contains the configuration values for the applications and charts to be deployed.
- `templates/`: Contains the Helm templates for generating Kubernetes manifests.

### Templates

- `_application.yaml`: Defines the template for creating ArgoCD `Application` resources for each application listed in the `values.yaml`.
- `_chart.yaml`: Defines the template for creating ArgoCD `Application` resources for each chart listed in the `values.yaml`.
- `application.yaml`: Includes the `_application.yaml` template.
- `chart.yaml`: Includes the `_chart.yaml` template.

### Values

The `values.yaml` file in the `app-of-apps` directory defines the applications and charts to be deployed:

```yaml
spec:
  destination:
    server: https://kubernetes.default.svc
  source:
    repoURL: https://github.com/GitTactician/app-of-apps
    targetRevision: main

applications:
  - name: argocd
    namespace: argocd

charts:
  - name: sealed-secrets
    namespace: kube-system
```

## ArgoCD

The root-application.yaml file defines the root ArgoCD Application resource that points to the app-of-apps chart:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: apps
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  destination:
    server: https://kubernetes.default.svc
  source:
    repoURL: https://github.com/GitTactician/app-of-apps
    targetRevision: main
    path: app-of-apps
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## Directory Structure

- `apps/`: Contains the ArgoCD application manifests.
- `base/`: Contains the base manifests for the applications.
- `overlays/`: Contains the overlay manifests for different environments (e.g., dev, prod).

## Usage

1. Install ArgoCD: Follow the ArgoCD installation guide to install ArgoCD in your Kubernetes cluster.

2. Deploy the Root Application: Apply the root-application.yaml file to create the root ArgoCD Application resource:

```bash
kubectl apply -f root-application.yaml
```

3. Monitor the Deployment: Use the ArgoCD UI or CLI to monitor the deployment of the applications and charts defined in the app-of-apps chart.
