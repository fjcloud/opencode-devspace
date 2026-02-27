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
3. Deploy to OpenShift with `oc apply -k deploy/overlays/dev`
4. Verify with `oc rollout status`, `oc logs`, `oc get routes`
5. Iterate — the same image tested in dev must be promoted to prod unchanged

## OpenShift conventions

- Use `oc` (not `kubectl`) — it has OpenShift-specific commands (routes, builds, etc.)
- Create a dedicated namespace per app: `oc new-project <name>` or use the current one
- Expose services with `oc expose svc/<name>` to create a Route
- Check cluster state before deploying: `oc project`, `oc status`
- For container builds on-cluster, prefer `oc new-build` or a Dockerfile BuildConfig

## Containerization

- Every app must have a `Containerfile` or `Dockerfile` in `src/`
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
    prod/
      kustomization.yaml
```

Deploy with `oc apply -k deploy/overlays/dev`.

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

## Resilience

When building HTTP services, implement:

- Timeouts on all outbound calls
- Retries with backoff for transient failures
- Circuit breakers to avoid cascading failures
- Rate limiting to protect against overload

Consider OpenShift Service Mesh for cross-cutting concerns without code changes.

## Observability

- Write logs to stdout/stderr — OpenShift collects them automatically
- Expose a `/metrics` endpoint (Prometheus format) for monitoring
- Expose a `/health` and `/ready` endpoint for probes
- Set up alerts on error rates, latency, and resource usage

## Security

- Stick to the restricted SCC — never request privileged or anyuid
- Use TLS between components if traffic carries sensitive data
- Never store secrets in images, env vars in Dockerfiles, or Git
- Scan images for vulnerabilities in CI before deploying

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
