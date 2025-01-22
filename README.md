# ArgoCD App of Apps

This repository demonstrates the "App of Apps" pattern using ArgoCD. The "App of Apps" pattern allows you to manage multiple applications with a single ArgoCD application, simplifying deployment and scaling workflows.

## Prerequisites

- Kubernetes cluster
- ArgoCD installed and configured
- `kubectl` configured to interact with your cluster

## Repository Structure

```
Directory structure:
└── gittactician-app-of-apps/
    ├── README.md
    ├── root-application.yaml
    ├── app-of-apps/
    │   ├── Chart.yaml
    │   ├── values.yaml
    │   ├── .helmignore
    │   └── templates/
    │       └── application.yaml
    └── apps/
        ├── apache-httpd/
        │   ├── Chart.yaml
        │   ├── values.yaml
        │   ├── .helmignore
        │   └── templates/
        │       └── deployment.yaml
        ├── argocd/
        │   ├── argocd.yaml
        │   └── kustomization.yaml
        ├── cert-manager/
        │   └── cert-manager.yaml
        ├── nginx-ingress/
        │   └── nginx-ingress.yaml
        └── sealed-secrets/
            ├── Chart.lock
            ├── Chart.yaml
            ├── values.yaml
            ├── .helmignore
            └── charts/
                └── sealed-secrets-2.12.0.tgz

```

### Main Components

- **app-of-apps/**: The main Helm chart that includes templates and values for deploying multiple applications.
- **apps/**: Directory containing individual application configurations for deployment.

## How It Works

### Helm Chart

The main Helm chart is located in the `app-of-apps/` directory and includes:

- `Chart.yaml`: Metadata for the chart.
- `values.yaml`: Configuration values for the applications to be deployed.
- `templates/`: Contains the template for generating ArgoCD Application resources (`application.yaml`).

### Application Template (`application.yaml`)

This Helm template generates ArgoCD `Application` resources based on the `values.yaml` file. It supports global and per-application overrides for critical parameters such as:

- `project`: Project name.
- `repoURL`: Repository URL for the application source.
- `targetRevision`: Branch, tag, or commit to deploy.
- `syncPolicy`: Configures automatic synchronization (optional).
- `prune`: Controls pruning of out-of-sync resources (optional).
- `selfHeal`: Enables automatic self-healing of resources (optional).
- `createNamespace`: Automatically creates namespaces for applications (optional).
- `destinationServer`: Kubernetes API server address specific to the application.

### Parameters in `values.yaml`

The `values.yaml` file defines the global and application-specific configurations.

#### Global Configuration

The following parameters apply globally unless overridden at the application level. Defaults are applied in the `application.yaml` template if not explicitly set:

- `spec.destination.server` (Required): Specifies the Kubernetes API server address.
- `spec.source.repoURL` (Required): Default Git repository URL.
- `spec.source.targetRevision` (Optional): Default branch, tag, or commit to deploy. Defaults to `main`.
- `spec.source.project` (optional): Default `default` project.
- `syncPolicy` (Optional): Enables global synchronization policy. Defaults to `false`.
- `prune` (Optional): Default pruning setting. Defaults to `true`.
- `selfHeal` (Optional): Default self-healing setting. Defaults to `true`.
- `createNamespace` (Optional): Default namespace creation setting. Defaults to `false`.

#### Application-Specific Configuration

Each application can override the global configuration using the following parameters:

- `name` (Required): Unique name for the application.
- `namespace` (Required): Target namespace for the application.
- `repoURL` (Optional): Application-specific repository URL. Overrides `spec.source.repoURL`.
- `targetRevision` (Optional): Application-specific branch, tag, or commit. Overrides `spec.source.targetRevision`.
- `project` (optional): Application-specific project. Overrides global `spec.source.project`.
- `syncPolicy` (Optional): Application-specific sync policy. Overrides global `syncPolicy`.
- `prune` (Optional): Application-specific pruning. Overrides global `prune`.
- `selfHeal` (Optional): Application-specific self-healing. Overrides global `selfHeal`.
- `createNamespace` (Optional): Application-specific namespace creation. Overrides global `createNamespace`.
- `destinationServer` (Optional): Application-specific Kubernetes API server. Overrides `spec.destination.server`.

### Example `values.yaml`

```yaml
spec:
  destination:
    server: https://kubernetes.default.svc
  source:
    repoURL: https://github.com/GitTactician/app-of-apps
    targetRevision: main

  # Global syncPolicy and namespace creation settings (optional)
  syncPolicy: false
  prune: true
  selfHeal: true
  createNamespace: false

applications:
  - name: sealed-secrets
    namespace: kube-system
    syncPolicy: true # Override global syncPolicy

  - name: nginx
    namespace: ingress-nginx
    repoURL: https://github.com/example/nginx-repo
    syncPolicy: true
    prune: false
    createNamespace: true

  - name: cert-manager
    namespace: cert-manager

  - name: apache-httpd
    namespace: apache-httpd
    prune: true
    selfHeal: true
    createNamespace: true

  - name: metrics-server
    namespace: kube-system
    repoURL: https://github.com/example/metrics-repo

  - name: strimzi-kafka-operator
    namespace: kafka
    repoURL: https://github.com/example/strimzi-repo
    appPath: helm/strimzi
    targetRevision: master
    syncPolicy: true
    prune: false
    createNamespace: true
    destinationServer: https://custom-api-server.example.com
```

### Key Points

- Applications inherit global settings unless overridden.
- The `destinationServer`, if specified for an application, overrides the global server.
- The `syncPolicy` can be configured globally or per application.
- Defaults applied in the `application.yaml` template include:
  - `spec.source.targetRevision`: Defaults to `main`.
  - `syncPolicy`: Defaults to `false`.
  - `prune`: Defaults to `false`.
  - `selfHeal`: Defaults to `false`.
  - `createNamespace`: Defaults to `false`.

## Usage

1. **Install ArgoCD**:
   Follow the [ArgoCD installation guide](https://argo-cd.readthedocs.io/en/stable/getting_started/) to install ArgoCD in your Kubernetes cluster.

2. **Deploy the Root Application**:
   Apply the `root-application.yaml` to create the root ArgoCD Application:

   ```bash
   kubectl apply -f root-application.yaml
   ```

3. **Monitor Deployment**:
   Use the ArgoCD UI or CLI to monitor the deployment of applications defined in the `app-of-apps` chart.

## Example Commands

- View ArgoCD Applications:

  ```bash
  argocd app list
  ```

- Sync Applications:

  ```bash
  argocd app sync <application-name>
  ```

- Check Application Status:

  ```bash
  argocd app get <application-name>
  ```
