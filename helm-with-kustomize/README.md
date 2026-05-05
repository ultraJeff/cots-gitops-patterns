# Approach 1: Helm with Kustomize (Helm is primary, Kustomize patches on top)

ArgoCD renders the Helm chart, then applies Kustomize patches as a post-renderer.
The ArgoCD Application uses **multi-source** — one source for the Helm chart, another
for the Kustomize overlays.

## How it works

1. ArgoCD pulls the Helm chart from the vendor's chart repo
2. ArgoCD pulls the Kustomize overlays from your Git repo
3. Helm renders the templates with your values
4. Kustomize patches the rendered output
5. ArgoCD applies the final result to the cluster

## Structure

```
helm-with-kustomize/
├── base/
│   └── values-ocp.yaml          # Helm values overrides for OpenShift
├── overlays/
│   └── ocp/
│       ├── kustomization.yaml   # Patches for SCC issues
│       └── patches/
│           └── fix-scc.yaml     # Strategic merge patch
└── argocd-application.yaml      # Multi-source ArgoCD Application
```

## When to use this approach

- You want ArgoCD to manage the Helm lifecycle (versions, releases)
- You need to overlay small patches on top of a third-party chart
- You want Helm and Kustomize concerns separated into different directories
