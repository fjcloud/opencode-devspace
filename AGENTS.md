# OpenCode on OpenShift Dev Spaces

You are running inside an OpenShift DevWorkspace container (Universal Developer Image).
The cluster is OpenShift 4.x on AWS, accessed via `oc` CLI (already authenticated).

## Project layout

- `src/` — all application source code lives here
- `deploy/` — Kubernetes/OpenShift manifests (Kustomize-ready, GitOps-compatible)
- `opencode.json` — OpenCode config (model, provider)
- `devfile.yaml` — DevWorkspace definition
- `AGENTS.md` — this file

Always create new code under `src/` and deployment manifests under `deploy/`. Keep the repo root clean.

## Development workflow

1. Write code in `src/`
2. Build and test locally inside the container before deploying
3. Deploy to OpenShift with `oc apply` from Kubernetes manifests
4. Verify with `oc get pods`, `oc logs`, `oc port-forward`

## OpenShift conventions

- Use `oc` (not `kubectl`) — it has OpenShift-specific commands (routes, builds, etc.)
- Create a dedicated namespace per app: `oc new-project <name>` or use the current one
- Expose services with `oc expose svc/<name>` to create a Route
- Check cluster state before deploying: `oc project`, `oc status`
- For container builds on-cluster, prefer `oc new-build` or a Dockerfile BuildConfig
- Always set resource requests/limits in Deployment manifests

## Containerization

- Every app must have a `Containerfile` or `Dockerfile` in `src/`
- Use multi-stage builds to keep images small
- Base on `registry.access.redhat.com/ubi9/ubi-minimal` or appropriate UBI images
- Never run as root — use `USER 1001` in the Containerfile
- Expose only necessary ports

## Kubernetes manifests

Place all manifests in `deploy/`. Structure for Kustomize:

```
deploy/
  base/           # shared manifests
    kustomization.yaml
    deployment.yaml
    service.yaml
    route.yaml
  overlays/       # per-environment overrides
    dev/
      kustomization.yaml
    prod/
      kustomization.yaml
```

Minimal resource set:

- `Deployment` with health probes (readiness + liveness)
- `Service` (ClusterIP)
- `Route` (for external access)
- `ConfigMap` / `Secret` for configuration (never hardcode secrets)

Use YAML, not JSON. Deploy with `oc apply -k deploy/overlays/dev`.

## Testing and validation

- Run unit tests locally before deploying: use the language's test runner
- After `oc apply`, verify the rollout: `oc rollout status deployment/<name>`
- Check pod logs for errors: `oc logs -f deployment/<name>`
- For HTTP services, test the route: `curl -s https://<route-hostname>/health`
- Clean up failed resources: `oc delete` what you created

## Code style

- Prefer simplicity and readability over cleverness
- One responsibility per file
- Meaningful names, no abbreviations
- Handle errors explicitly — don't silently ignore them
- Comment only what the code cannot express on its own
