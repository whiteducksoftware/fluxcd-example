# fluxcd-example

Sample configuration repository for Fluxcd thats install the following:

- ingress-nginx
- cert-manager
- demo application

## Repository structure

This Git repository contains the following top directories:

- **apps** dir contains workloads for cluster users
- **charts** dir contains local Helm charts
- **clusters** dir contains the Flux configuration per cluster
- **infra** dir contains common infra tools, Helm repository definitions and cluster default settings

```shell
./
├── clusters
│   ├── dev
│   ├── prod
└── charts
│   ├── ...
└── infra
│   ├── base
│   ├── config
│   ├── dev
│   └── prod
└── apps
    ├── base
    ├── dev
    └── prod
```

### clusters

The `clusters` dir contains `Kustomizations` objects that define the source of Kubernetes manifests:

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1beta1
kind: Kustomization
metadata:
  name: defaults
  namespace: flux-system
spec:
  interval: 5m
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: ./infra/defaults
  prune: true
  validation: client
```

### infra

The `infra` configuration is structured into:

- `infra/base` dir contains Helm release definitions
- `infra/config` dir contains RBAC config and other infra config
- `infra/dev` dir contains the development Helm release values
- `infra/prd` dir contains the production Helm release values

```shell
./infra/
├── base
│   ├── ingress-nginx
│   │   ├── kustomization.yaml
│   │   ├── namespace.yaml
│   │   └── release.yaml
│   └── ...
├── defaults
│   ├── kustomization.yaml
│   ├── rbac.yaml
│   └── sources.yaml
├── dev
│   ├── ingress-nginx
│   │   ├── kustomization.yaml
│   │   └── values.yaml
│   └── ...
└── prd
    ├── ingress-nginx
    │   ├── kustomization.yaml
    │   └── values.yaml
    └── ...
```

The `infra/base/` dir contains HelmReleases as base for all clusters:

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: ingress-nginx
spec:
  chart:
    spec:
      chart: ingress-nginx
      version: 4.0.17
      sourceRef:
        kind: HelmRepository
        name: ingress-nginx
        namespace: flux-system
  values:
  # add values here
```

The `infra/defaults/` dir contains namespaces, RBAC and Helm repository definitions:

```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta1
kind: HelmRepository
metadata:
  name: ingress-nginx
  namespace: flux-system
spec:
  interval: 10m0s
  url: https://kubernetes.github.io/ingress-nginx
```

The `infra/defaults/dev` & `infra/defaults/prd` dir containts Kustomize patches with the environments specific values:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - ../../base/ingress-nginx
patchesStrategicMerge:
  - values.yaml
```

### apps

The `apps` dir contains user workloads with a similuar structure like we have with the `infra` dir:

```shell
./apps/
├── base
│   ├── demo
│   │   ├── kustomization.yaml
│   │   ├── namespace.yaml
│   │   └── release.yaml
│   └── ...
├── dev
│   ├── demo
│   │   ├── kustomization.yaml
│   │   └── values.yaml
│   └── ...
└── prd
    ├── demo
    │   ├── kustomization.yaml
    │   └── values.yaml
    └── ...
```
