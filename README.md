# OpenCode DevSpace

## Prerequisites

### 0. Cluster sizing (ROSA)

The stack requires sufficient CPU/memory workers plus a GPU node:

- **Workers**: 5x `m7i.xlarge` (4 vCPU, 16 GiB each) — or fewer larger nodes (e.g. 3x `m7i.2xlarge`). 3 small workers is not sufficient for the full operator stack.
- **GPU**: 1x `g6e.xlarge` (NVIDIA L40S) — tainted to reserve it for inference only.

```bash
rosa create machine-pool -c <cluster-name> \
  --name gpu \
  --replicas 1 \
  --instance-type g6e.xlarge \
  --availability-zone <az> \
  --taints 'nvidia.com/gpu=present:NoSchedule'
```

Wait for all nodes to be ready:

```bash
oc wait node -l node.kubernetes.io/instance-type=g6e.xlarge --for=condition=Ready --timeout=600s
```

### 1. Install operators

```bash
oc apply -k https://github.com/fjcloud/opencode-devspace/deploy/prereq/operator
```

Wait for all operators to be ready:

```bash
oc wait deployment/devspaces-operator -n openshift-operators --for=condition=Available --timeout=300s
oc wait deployment/nfd-controller-manager -n openshift-nfd --for=condition=Available --timeout=300s
oc wait deployment/gpu-operator -n nvidia-gpu-operator --for=condition=Available --timeout=300s
oc wait deployment/rhods-operator -n redhat-ods-operator --for=condition=Available --timeout=300s
```

### 2. Create instances

```bash
oc apply -k https://github.com/fjcloud/opencode-devspace/deploy/prereq/instance
```

Wait for instances to be ready:

```bash
oc wait checluster/devspaces -n openshift-operators --for=jsonpath='{.status.chePhase}'=Active --timeout=300s
oc wait nodefeaturediscovery/nfd-instance -n openshift-nfd --for=condition=Available --timeout=300s
oc wait clusterpolicy/gpu-cluster-policy --for=condition=Ready --timeout=600s
oc wait datasciencecluster/default-dsc --for=condition=Ready --timeout=300s
```

### 3. Deploy the LLM inference service

```bash
oc apply -k https://github.com/fjcloud/opencode-devspace/deploy/prereq/inference
```

Wait for the model to be ready (first start downloads the model weights):

```bash
oc wait inferenceservice/qwen35 -n llm-inference --for=condition=Ready --timeout=900s
```

## Launch

Get your workspace URL:

```bash
echo "$(oc get checluster devspaces -n openshift-operators -o jsonpath='{.status.cheURL}')/#https://github.com/fjcloud/opencode-devspace"
```

Open the resulting link in your browser.

## Getting started

On first launch, close the VS Code welcome tab — this only happens once, the preference is persisted on your workspace PVC.

OpenCode starts automatically in the terminal. Try a prompt like:

> Build me a hello-world Go app

OpenCode uses the `AGENTS.md` file as context, so it will scaffold the application following the project conventions: namespace layout (`<app>-build`, `-dev`, `-stage`, `-prod`), Kustomize manifests under `deploy/`, on-cluster builds via `oc new-build`, edge TLS routes, and all the OpenShift best practices defined in the repo.
