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

# ArgoCD Helm Chart with App-of-Apps Pattern

This Helm chart is designed to automate the creation and management of ArgoCD applications using the "App-of-Apps" pattern. It allows for centralized management of multiple applications with a global configuration that can be overridden at the application level.

## Features

- Centralized global defaults for common configurations.
- Per-application overrides for fine-grained control.
- Automated configuration of application `spec` fields such as `source`, `destination`, `syncPolicy`, `metadata`, and more.
- Support for multiple sources in a single application.
- Fallback mechanism for default values when specific values are not provided.
- Flexible configuration via `values.yaml`.

## Default Fields

### Global Defaults

The following fields are defined globally in `values.yaml` and apply to all applications unless overridden at the application level:

- **destination**: Specifies the target cluster and namespace.
  - Default:
    ```yaml
    destination:
      server: https://kubernetes.default.svc
      namespace: argocd
    ```
- **source**: Defines the repository and path for application manifests.
  - Default:
    ```yaml
    source:
      repoURL: "https://example.com/repo.git"
      targetRevision: "main"
      path: "apps"
    ```
- **syncPolicy**: Controls the synchronization behavior of applications.
  - Default:
    ```yaml
    syncPolicy:
      automated:
        prune: true
        selfHeal: true
    ```
- **metadata**: Provides default metadata for applications.
  - Default:
    ```yaml
    metadata:
      namespace: argocd
    ```
- **ignoreDifferences**: Specifies resource fields to ignore during diffing.
  - Default: None (optional field).

### Application-Level Overrides

Each application defined in the `applications` section of `values.yaml` can override the global defaults. Overrides apply only to the specific application and take precedence over global settings.

#### Example Application-Specific Overrides:

```yaml
applications:
  - name: example-app
    namespace: custom-namespace
    destination:
      server: https://custom-cluster.svc
      namespace: custom-namespace
    source:
      repoURL: "https://custom-repo.com/app.git"
      path: "custom/path"
      targetRevision: "develop"
    syncPolicy:
      automated:
        prune: false
        selfHeal: true
    metadata:
      annotations:
        custom-annotation: "value"
    ignoreDifferences:
      - group: "apps"
        kind: "Deployment"
        jsonPointers:
          - "/spec/replicas"
```

## Fallback Mechanism

The chart employs a two-stage fallback mechanism for default values:

1. **Application-Level Check**: If a field is specified for an application, its value is used.
2. **Global Default Check**: If a field is not specified for an application, the corresponding global default value is used. If no global default exists, the field remains unset (if optional) or triggers an error (if required).

### Example

Given the following `values.yaml`:

```yaml
spec:
  destination:
    server: https://global-cluster.svc
    namespace: global-namespace
  source:
    repoURL: "https://global-repo.com/repo.git"
    targetRevision: "main"
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
  metadata:
    namespace: global-namespace

applications:
  - name: app1
    namespace: app1-namespace
    source:
      path: "app1/path"
  - name: app2
    destination:
      namespace: app2-namespace
```

- **`app1`**:

  - `destination.server`: Fallback to global (`https://global-cluster.svc`).
  - `destination.namespace`: Use application-specific (`app1-namespace`).
  - `source.path`: Use application-specific (`app1/path`).
  - `source.repoURL` and `targetRevision`: Fallback to global.
  - `syncPolicy` and `metadata`: Fallback to global.

- **`app2`**:
  - `destination.server`: Fallback to global (`https://global-cluster.svc`).
  - `destination.namespace`: Use application-specific (`app2-namespace`).
  - `source`: Fully fallback to global.
  - `syncPolicy` and `metadata`: Fallback to global.

## Example `values.yaml`

```yaml
spec:
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  source:
    repoURL: "https://github.com/example/repo.git"
    targetRevision: "main"
    path: "apps"
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
  metadata:
    namespace: argocd

applications:
  - name: app1
    namespace: custom-namespace
    source:
      path: "custom/path"
  - name: app2
    destination:
      namespace: another-namespace
  - name: app3
    syncPolicy:
      automated:
        prune: false
        selfHeal: true
```

This setup ensures consistency across applications while allowing for customization as needed.

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
