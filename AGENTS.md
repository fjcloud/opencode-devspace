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
2. Build and test locally with `podman build` / `podman run` — podman is available in the container, use it instead of installing extra tooling
3. Deploy to OpenShift with `oc apply -k deploy/overlays/dev`
4. Verify with `oc rollout status`, `oc logs`, `oc get routes`
5. Iterate — the same image tested in dev must be promoted to prod unchanged

## Namespace strategy

Four namespaces per application:

| Namespace | Purpose |
|-----------|---------|
| `<app>-build` | BuildConfig + ImageStream. Images are built and pushed here. |
| `<app>-dev` | Development environment. Pulls images from `<app>-build`. |
| `<app>-stage` | Staging / QA. Pulls images from `<app>-build`. |
| `<app>-prod` | Production. Pulls images from `<app>-build`. |

Setup:

```bash
oc new-project <app>-build
oc new-project <app>-dev
oc new-project <app>-stage
oc new-project <app>-prod

# Grant each environment pull access to the build registry
for ns in <app>-dev <app>-stage <app>-prod; do
  oc policy add-role-to-user system:image-puller system:serviceaccount:${ns}:default -n <app>-build
done
```

Every Deployment MUST use the `image.openshift.io/triggers` annotation to auto-deploy when an ImageStream tag is updated:

```yaml
metadata:
  annotations:
    image.openshift.io/triggers: >-
      [{"from":{"kind":"ImageStreamTag","name":"<image>:<tag>","namespace":"<app>-build"},
        "fieldPath":"spec.template.spec.containers[?(@.name==\"<container>\")].image"}]
```

This way the image field is automatically resolved and updated by OpenShift — no hardcoded registry URLs.

Promotion is done by tagging, not rebuilding:

```bash
oc tag <app>-build/<image>:latest <app>-build/<image>:stage
oc tag <app>-build/<image>:stage  <app>-build/<image>:prod
```

When a tag is updated, the trigger annotation rolls out the new image automatically.

## OpenShift conventions

- Use `oc` (not `kubectl`) — it has OpenShift-specific commands (routes, builds, etc.)
- Always create edge-terminated TLS Routes: `oc create route edge <name> --service=<svc>` (never plain HTTP)
- Check cluster state before deploying: `oc project`, `oc status`
- For container builds on-cluster, prefer `oc new-build` or a Dockerfile BuildConfig

## Containerization

- Every app must have a `Dockerfile` in `src/`
- Use multi-stage builds: separate build image from runtime image to reduce attack surface
- Base on `registry.access.redhat.com/ubi9/ubi-minimal` or appropriate UBI images — trusted, tested, supported
- Always use the latest UBI tag to include all security fixes
- Never run as root — use `USER 1001` and support arbitrary user IDs (OpenShift restricted SCC)
- Expose only necessary ports
- One process per container — no supervisor hacks

## Kubernetes manifests

Place all manifests in `deploy/`. Structure for Kustomize:

```
deploy/
  base/
    kustomization.yaml
    deployment.yaml
    service.yaml
    route.yaml
  overlays/
    dev/
      kustomization.yaml
    stage/
      kustomization.yaml
    prod/
      kustomization.yaml
```

Deploy per environment:

```bash
oc apply -k deploy/overlays/dev   -n <app>-dev
oc apply -k deploy/overlays/stage -n <app>-stage
oc apply -k deploy/overlays/prod  -n <app>-prod
```

## Manifest requirements

Every Deployment MUST include:

- **Resource requests and limits** for CPU and memory on every container
- **Liveness probe** — restarts the pod if the app is stuck
- **Readiness probe** — stops routing traffic until the app is ready
- **Graceful shutdown** — handle SIGTERM, drain in-flight requests, set `terminationGracePeriodSeconds`
- **PodDisruptionBudget** — protect availability during node maintenance and cluster scaling

Configuration MUST be external to the image:

- Use `ConfigMap` for non-sensitive config
- Use `Secret` for credentials (never hardcode)
- The same image is promoted from dev to prod — only config changes per environment

## Observability

- Write logs to stdout/stderr — OpenShift collects them automatically
- Expose a `/metrics` endpoint (Prometheus format) for monitoring
- Expose a `/health` and `/ready` endpoint for probes
- Set up alerts on error rates, latency, and resource usage

## Testing and validation

- Run unit tests locally before deploying: use the language's test runner
- After `oc apply`, verify: `oc rollout status deployment/<name>`
- Check pod logs: `oc logs -f deployment/<name>`
- Test the route: `curl -s https://<route-hostname>/health`
- Clean up failed resources: `oc delete` what you created

## Code style

- Prefer simplicity and readability over cleverness
- One responsibility per file
- Meaningful names, no abbreviations
- Handle errors explicitly — don't silently ignore them
- Comment only what the code cannot express on its own
