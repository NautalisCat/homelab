# Homelab

A personal Kubernetes homelab running on HP ProDesk 400 G4 (soon to be a cluster) managed entirely through GitOps using Flux CD. This repository is the single source of truth for everything running in the cluster. If it's not in Git, it doesn't exist on the cluster.

## Infrastructure

| Node | Role | Hardware |
|------|------|----------|
| Hades | Control Plane | HP ProDesk 400 G4, Intel Core i5 8000 series |
| Persephone | Worker | — |
| Minthe | Worker | — |

## GitOps with Flux CD

This homelab uses [Flux CD v2](https://fluxcd.io/) to implement a full GitOps workflow. Flux watches this repository and automatically reconciles the cluster state to match what is defined here. Every change to the cluster goes through Git — no manual `kubectl apply` commands.

## Repository Structure

```
homelab/
├── apps/
│   ├── base/               # Core app definitions (deployment, namespace, config)
│   │   ├── linkding/
│   │   └── cloudflared/
│   └── staging/            # Environment overlays and encrypted secrets
│       ├── linkding/
│       └── cloudflared/
├── monitoring/
│   └── controllers/
│       ├── base/           # Helm chart definitions
│       └── staging/        # Encrypted secrets and overrides
└── clusters/
    └── staging/
        ├── apps.yaml               # Flux Kustomization for apps
        ├── monitoring.yaml         # Flux Kustomization for monitoring
        └── flux-system/            # Flux installation manifests
```

## Secrets Management

Secrets are managed using [SOPS](https://github.com/getsops/sops) with [age](https://github.com/FiloSottile/age) encryption. Sensitive values are encrypted before being committed to Git, meaning secrets live alongside the rest of the infrastructure code without exposing credentials. Flux automatically decrypts secrets at apply time using a private key stored as a cluster secret.

## Applications

### Linkding
Self-hosted bookmark manager. Runs as a non-root user with a hardened security context, persistent storage via a PVC, and superuser credentials managed through SOPS encrypted secrets.

### Cloudflare Tunnel
All external traffic is routed through a Cloudflare tunnel — no ports are exposed on the host machine and the server's IP is never public. Traffic is encrypted end to end.

### Monitoring (kube-prometheus-stack)
Prometheus and Grafana deployed via Helm and managed by Flux. Grafana is accessible externally through the Cloudflare tunnel with admin credentials stored as SOPS encrypted secrets.
