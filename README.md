# Repository Structure

```text
fleet-repo
│
├── README.md
│
├── addons/                           # Platform addon configurations per cluster
│   ├── aks-eastus/
│   ├── aks-southeastasia/
│   │   └── values.yaml               # AKS addon values (Prometheus, metrics, etc.)
│   ├── base/
│   │   ├── Chart.yaml                # Base Helm chart
│   │   └── values.yaml               # Shared addon configuration
│   ├── eks-us-east-1/
│   ├── eks-us-west-2/
│   │   └── values.yaml               # EKS addon values
│   └── gke-europe-west1/
│       └── values.yaml               # GKE addon values
│
├── apps/                             # GitOps application definitions
│   ├── applicationsets/
│   │   ├── platform-addons.yaml      # Deploys platform addons to all clusters
│   │   └── workloads.yaml            # Deploys workloads to all clusters
│   └── base/
│       └── nginx/
│           └── nginx.yaml            # Sample workload
│
├── clusters/                         # Cluster registration manifests
│   ├── aks-eastus/
│   ├── aks-southeastasia/
│   │   └── cluster.yaml
│   ├── eks-us-east-1/
│   ├── eks-us-west-2/
│   │   └── cluster.yaml
│   └── gke-europe-west1/
│       └── cluster.yaml
│
├── hub/                              # Hub cluster components
│   ├── argocd/
│   │   ├── app-of-apps.yaml          # Root GitOps application
│   │   ├── crossplane-app.yaml       # Crossplane deployment via ArgoCD
│   │   ├── kustomization.yaml
│   │   └── values.yaml
│   │
│   ├── crossplane/
│   │   ├── compositions/
│   │   │   ├── xdatabase-aws.yaml    # AWS database composition
│   │   │   ├── xdatabase-gcp.yaml    # GCP database composition
│   │   │   └── xdatabase-xrd.yaml    # Composite resource definition
│   │   │
│   │   ├── install.yaml              # Crossplane installation
│   │   ├── kustomization.yaml
│   │   └── providers/
│   │       ├── aws-providerconfig.yaml
│   │       ├── azure-providerconfig.yaml
│   │       ├── gcp-providerconfig.yaml
│   │       ├── providers.yaml
│   │       └── runtimeconfig.yaml
│   │
│   └── thanos/
│       └── install.yaml              # Centralized monitoring stack
│
├── infra/
│   └── claims/
│       ├── app-namespace.yaml        # Namespace claim
│       └── orders-db.yaml            # Database claim
│
├── spoke-clusters/
│   └── spoke-eks.yaml                # Spoke cluster bootstrap
│
├── test/
│   └── s3-bucket.yaml                # Crossplane testing resource
│
├── test-argocd/
│   └── nginx.yaml                    # ArgoCD deployment test
│
├── test-argocd-app.yaml              # Test ArgoCD application
│
└── hub-cluster.yaml                  # Hub cluster creation manifest
```

## Key Components

### Hub Cluster

The AWS EKS hub cluster hosts:

* ArgoCD
* Crossplane
* Thanos Query
* Grafana

This cluster acts as the central control plane for the entire fleet.

### Spoke Clusters

The platform manages three spoke clusters:

* AWS EKS (us-west-2)
* Azure AKS (southeastasia)
* Google GKE (europe-west1)

Application deployments are automatically propagated to all spokes through ArgoCD ApplicationSets.

### Crossplane

Crossplane provides Infrastructure-as-Code through Kubernetes APIs and enables provisioning of:

* AWS resources
* Azure resources
* Google Cloud resources

using Kubernetes Custom Resources.

### Monitoring Stack

Each spoke cluster runs Prometheus.

Metrics flow:

```text
Prometheus (Spokes)
        ↓
Thanos Receive
        ↓
Thanos Query
        ↓
Grafana
```

providing a unified multi-cloud monitoring experience.

### GitOps Workflow

```text
Developer
    ↓
Git Push
    ↓
GitHub Repository
    ↓
ArgoCD
    ↓
ApplicationSets
    ↓
EKS / AKS / GKE
```

All infrastructure and application changes are performed through Git commits.
