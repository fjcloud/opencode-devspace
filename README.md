# OpenCode DevSpace

Get your workspace URL by running:

```bash
oc get checluster devspaces -n openshift-operators -o jsonpath='{.status.cheURL}' | xargs -I{} echo "{}/#https://github.com/fjcloud/opencode-devspace"
```

Open the resulting link in your browser to launch the workspace.
