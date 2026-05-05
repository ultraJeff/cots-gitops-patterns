# Approach 2: Kustomize with Helm (Kustomize is primary, Helm chart pulled in)

Kustomize is the top-level tool. It pulls in the Helm chart via the `helmCharts`
field in `kustomization.yaml`, renders it, and applies patches — all before ArgoCD
ever sees the manifests.

## How it works

1. ArgoCD sees a Kustomize source (your Git repo)
2. Kustomize reads `kustomization.yaml`, finds the `helmCharts` field
3. Kustomize pulls and renders the Helm chart with your values
4. Kustomize applies patches to the rendered output
5. ArgoCD applies the final result to the cluster

## Structure

```
kustomize-with-helm/
├── kustomization.yaml           # Pulls Helm chart + applies patches
├── values-ocp.yaml              # Helm values for OpenShift
├── scc/
│   ├── cots-app-scc.yaml        # Custom SCC for the COTS app
│   └── scc-rolebinding.yaml     # Bind SCC to app service accounts
└── argocd-application.yaml      # Simple single-source ArgoCD Application
```

## When to use this approach

- You want a single `kustomization.yaml` that defines everything
- You prefer Kustomize as the "outer" tool that owns the full picture
- Simpler ArgoCD Application (single source, not multi-source)
- Easier to reason about for teams already comfortable with Kustomize

## SCC vs Deployment patches — do you need both?

This example includes both a custom SCC (with RoleBindings) and Kustomize patches
on the Deployments. They solve different problems and you may only need one depending
on what the chart actually does wrong.

**SCC + RoleBinding only** — Use this when the chart's pod specs are fine but
OpenShift's default `restricted` SCC rejects them. Common case: the container image
runs as a specific UID that doesn't match OpenShift's assigned UID range. The SCC
grants permission (e.g., `RunAsAny`), and the pods start without touching the
Deployments at all. This is the simpler fix and the one to try first.

**Deployment patches only** — Use this when the chart explicitly sets something
problematic in the pod spec, like `privileged: true` or `allowPrivilegeEscalation: true`.
No SCC can fix a pod that's actively requesting dangerous settings — you need to
patch those out of the Deployment spec. You'd then rely on an existing SCC (like
`nonroot` or `restricted`) rather than creating a custom one.

**Both** — Use this when you're dealing with a mix: some pods just need a wider SCC
to run as their expected UID, while others have explicitly bad security settings in
their specs that need to be patched out. Start with the SCC approach, see which pods
still fail, and add targeted patches for those.

## Requirements

- ArgoCD's Kustomize must have Helm support enabled (`--enable-helm`)
- Recent ArgoCD versions (2.6+) have this on by default
