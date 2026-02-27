# OpenCode DevSpace

## Prerequisites

Install the Dev Spaces operator and create the instance:

```bash
# 1. Install the operator
oc apply -k https://github.com/fjcloud/opencode-devspace/deploy/prereq/operator

# 2. Wait for the CRD to be available
oc wait crd/checlusters.org.eclipse.che --for=condition=Established --timeout=300s

# 3. Create the instance
oc apply -k https://github.com/fjcloud/opencode-devspace/deploy/prereq/instance

# 4. Wait for the instance to be ready
oc wait checluster/devspaces -n openshift-operators --for=jsonpath='{.status.chePhase}'=Active --timeout=300s
```

## Launch

Get your workspace URL:

```bash
oc get checluster devspaces -n openshift-operators -o jsonpath='{.status.cheURL}' | xargs -I{} echo "{}/#https://github.com/fjcloud/opencode-devspace"
```

Open the resulting link in your browser.
