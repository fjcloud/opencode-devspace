# OpenCode DevSpace

## Prerequisites

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
oc wait datasciencecluster/default-dsc --for=condition=Available --timeout=300s
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
