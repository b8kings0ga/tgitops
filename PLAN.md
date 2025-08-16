# GitOps Architectural Plan for `tgitops`

This document outlines the architectural plan for the `tgitops` repository, which serves as the central configuration hub for a multi-environment, multi-region GitOps workflow.

## 1. Repository Structure

The repository is organized into several key directories, each with a distinct purpose. This structure is designed to provide a clear separation of concerns between environment-specific configurations, application definitions, infrastructure resources, and reusable components.

```
.
├── envs/
│   ├── dev/
│   │   └── us-east-1/
│   │       └── my-app-a/
│   │           └── values.yaml
│   └── prod/
│       └── us-east-1/
│           └── my-app-a/
│               ├── app.yaml
│               └── values.yaml
├── appsets/
│   ├── dev/
│   │   ├── services.yaml
│   │   └── crossplane.yaml
│   └── prod/
│       └── services.yaml
├── crossplane/
│   └── databases/
│       └── my-app-a-db.yaml
├── vclusters/
│   ├── dev-vcluster-manifest.yaml
│   └── prod-vcluster-manifest.yaml
├── charts/
│   └── golden-charts/
│       └── Chart.yaml
└── README.md
```

## 2. Directory Breakdown and Workflow

### 2.1 `charts/`

*   **Purpose**: This directory holds reusable Helm charts.
*   **`golden-charts/`**: Contains a common base chart for all applications. It is a standard Helm chart with minimal, sensible defaults, designed to be heavily customized through `values.yaml` files. This promotes consistency and reduces boilerplate across services.

### 2.2 `envs/`

*   **Purpose**: This is the core of environment-specific configuration. It contains the overrides and specific settings for each application in each environment and region.
*   **Structure**: The hierarchy is `envs/<environment>/<region>/<application>/`.
*   **`values.yaml`**: This file contains the Helm values that will be applied to the `golden-charts` for a specific application in a specific environment. It overrides the default chart values with settings for replicas, image tags, resource limits, etc.
*   **`app.yaml`**: (Primarily for `prod` or complex environments) This file holds variables for the Argo CD ApplicationSet templates, allowing for dynamic application generation based on environment-specific parameters.

### 2.3 `appsets/`

*   **Purpose**: This directory contains the Argo CD ApplicationSet definitions. ApplicationSets are used to automatically generate Argo CD Applications based on generators that scan the `envs/` directory.
*   **Workflow**:
    1.  An ApplicationSet in this directory (e.g., `appsets/dev/services.yaml`) uses a Git generator to look for `values.yaml` files under a specific path (e.g., `envs/dev/**`).
    2.  For each match, it generates an Argo CD Application.
    3.  The generated Application points to the `golden-charts` in the `charts/` directory and uses the corresponding `values.yaml` from the `envs/` directory as its input.
*   **Separation**: Different ApplicationSets can manage different types of resources. For example, `services.yaml` manages application deployments, while `crossplane.yaml` could manage infrastructure resources.

### 2.4 `vclusters/`

*   **Purpose**: Defines the virtual Kubernetes clusters (vclusters) where applications will be deployed.
*   **Strategy**: Instead of deploying directly to a large, shared Kubernetes cluster, each application or environment runs in an isolated vcluster. This improves multi-tenancy, security, and resource management. The manifests in this directory are used to provision these vclusters.

### 2.5 `crossplane/`

*   **Purpose**: Manages cloud infrastructure resources using Crossplane.
*   **Workflow**: This directory contains Crossplane compositions and claims for resources like databases (e.g., `my-app-a-db.yaml`), caches, or message queues.
*   **Integration**: An ApplicationSet (like `appsets/dev/crossplane.yaml`) can be configured to discover and apply these manifests, allowing infrastructure to be provisioned and managed through the same GitOps workflow as applications.

## 3. End-to-End Deployment Flow

1.  **Code Change**: A developer pushes a code change to an application repository (e.g., `tapp`).
2.  **CI Pipeline**: The CI pipeline builds a new container image and pushes it to a registry.
3.  **Configuration Update**: The CI pipeline checks out the `tgitops` repository and updates the `image.tag` in the appropriate `envs/.../values.yaml` file.
4.  **Git Push**: The change is committed and pushed to the `tgitops` repository.
5.  **Argo CD Sync**:
    *   Argo CD's ApplicationSet controller detects the change in the `envs/` directory.
    *   It updates the corresponding Argo CD Application with the new Helm values.
    *   The Argo CD Application controller syncs the changes, applying the updated manifests (generated from the `golden-charts` and the new `values.yaml`) to the target `vcluster`.
6.  **Deployment**: The Kubernetes deployment in the vcluster performs a rolling update to the new container image version.