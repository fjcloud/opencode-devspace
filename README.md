# OpenCode DevSpace

```bash
oc get checluster devspaces -n openshift-operators -o jsonpath='{.status.cheURL}' | xargs -I{} echo "{}/#https://github.com/fjcloud/opencode-devspace"
```
