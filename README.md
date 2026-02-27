# OpenCode DevSpace

## Prerequisites

Install the Dev Spaces operator and create the instance:

```bash
oc apply -k https://github.com/fjcloud/opencode-devspace/deploy/prereq/operator
oc apply -k https://github.com/fjcloud/opencode-devspace/deploy/prereq/instance
```

Wait for the instance to be ready:

```bash
oc wait checluster/devspaces -n openshift-operators --for=jsonpath='{.status.chePhase}'=Active --timeout=300s
```

## Launch

Get your workspace URL:

```bash
oc get checluster devspaces -n openshift-operators -o jsonpath='{.status.cheURL}' | xargs -I{} echo "{}/#https://github.com/fjcloud/opencode-devspace"
```

Open the resulting link in your browser.
