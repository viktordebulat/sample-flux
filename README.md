# Sample Demo FluxCD

This is a demo cluster infrastructure, which is using Flux CD to follow Gitops approach, and other fancy tools. Feel free to play with those samples or use it as runbook to get familiar with basics.

## Repository structure

```
.
├── README.md
├── apps                     # Project apps manifests
├── apps-dev                 # Development project apps manifests 
├── flux-system              # Flux control manifests
|   ├── namespaces              # Namespaces
|   └── helmrepos               # Helm Repositories definitions
├── kube-system              # Kube-system apps manifests
└── supp                     # Supplementary applications
```


## GitOps flow

- Flux stack runs inside k8s cluster
- Flux stack apps runs async control loop, checking for various things:
  - Git repositories update.
  - Docker image registries update.
  - Helm repositores updates.
- Flux stack apps reacts on updates and perform k8s cluster update according to source updates in `main` branch.

### Bootstrap Flux

Create Github Classic Token first with full repo level access.

```
export GITHUB_TOKEN=<my-token>

flux bootstrap github --owner=<user> --repository=<repository name> --private=false --personal=true --path=cluster
```
