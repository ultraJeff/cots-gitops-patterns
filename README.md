# COTS on OpenShift with GitOps

Patterns for deploying commercial off-the-shelf (COTS) applications on OpenShift
using ArgoCD when the vendor's Helm chart wasn't written with OpenShift in mind.

The most common issue: the chart's pods fail with SCC-related errors because the
chart assumes vanilla Kubernetes security defaults.

## Two approaches

### [Approach 1: Helm with Kustomize](helm-with-kustomize/)
Helm is the primary source. ArgoCD uses multi-source to render the Helm chart and
apply Kustomize patches on top. Use this when you want ArgoCD to manage the Helm
chart lifecycle directly.

### [Approach 2: Kustomize with Helm](kustomize-with-helm/)
Kustomize is the primary source. It pulls in the Helm chart via `helmCharts`,
renders it, and applies patches — all in one `kustomization.yaml`. Simpler ArgoCD
Application config (single source).

## Which one should I use?

Both produce the same result on the cluster. The difference is who owns the outer
loop:

| | Helm with Kustomize | Kustomize with Helm |
|---|---|---|
| ArgoCD source type | Multi-source | Single source (Kustomize) |
| Helm lifecycle | ArgoCD manages it | Kustomize manages it |
| Complexity | More moving parts | One file owns everything |
| ArgoCD Helm visibility | Full (chart version, values) | None (just sees Kustomize) |
| Best for | Teams that think in Helm first | Teams that think in Kustomize first |
