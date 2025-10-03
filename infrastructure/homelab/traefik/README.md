Traefik (Ingress Controller) for homelab

This folder contains Flux resources to install Traefik using the official Helm chart via Flux HelmRelease.

Files:

- `namespace.yaml` - namespace to install Traefik into (`traefik`).
- `helmrepository.yaml` - Flux HelmRepository for `helm.traefik.io/traefik`.
- `helmrelease.yaml` - HelmRelease to install Traefik with sensible defaults.

Notes and next steps:

- By default the chart is configured to create a Service of type `LoadBalancer`. If your cluster doesn't provide an external LB (bare-metal), change `service.type` to `NodePort` or configure a MetalLB installation.
- TLS: to use ACME certificates, install `cert-manager` and enable `ingressRoute` TLS options in the HelmRelease `values` or provide a `values.yaml` override.
- Dashboard: disabled by default. To enable, set `dashboard.enabled: true` in the HelmRelease values and secure access (e.g. RBAC + ingress with auth).
