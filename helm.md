# Helm Cheat Sheet

## Repos
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm repo list
helm repo remove bitnami
helm search repo nginx                  # search added repos
helm search hub wordpress                 # search Artifact Hub
```

## Installing & upgrading
```bash
helm install myrelease bitnami/nginx
helm install myrelease bitnami/nginx --namespace web --create-namespace
helm install myrelease bitnami/nginx -f values.yaml
helm install myrelease bitnami/nginx --set replicaCount=3
helm upgrade myrelease bitnami/nginx -f values.yaml
helm upgrade --install myrelease bitnami/nginx     # install if not present, upgrade if present
helm upgrade myrelease bitnami/nginx --set image.tag=1.25
helm rollback myrelease 1                             # roll back to revision 1
helm uninstall myrelease
```

## Inspecting releases
```bash
helm list                            # releases in current namespace
helm list -A                           # releases across all namespaces
helm status myrelease
helm history myrelease                    # revision history
helm get values myrelease                   # values currently in use
helm get manifest myrelease                   # rendered k8s manifests for a release
helm get notes myrelease                         # post-install notes
```

## Chart development
```bash
helm create mychart                    # scaffold a new chart
helm lint mychart/                       # validate chart structure
helm template mychart/                     # render templates locally (no cluster needed)
helm template mychart/ -f values.yaml
helm package mychart/                        # package into a .tgz
helm dependency update mychart/                # fetch chart dependencies
helm show values bitnami/nginx                   # view a chart's default values.yaml
helm show chart bitnami/nginx                      # view Chart.yaml metadata
helm pull bitnami/nginx --untar                      # download + extract a chart locally
```

## Dry runs & debugging
```bash
helm install myrelease bitnami/nginx --dry-run --debug
helm upgrade myrelease bitnami/nginx --dry-run
helm diff upgrade myrelease bitnami/nginx        # requires helm-diff plugin
helm test myrelease                                # run chart's test hooks
```

## Plugins
```bash
helm plugin list
helm plugin install https://github.com/databus23/helm-diff
```

## Useful one-liners
```bash
helm list -A -o json | jq '.[] | .name'                    # release names, scriptable
helm get values myrelease -a                                  # all values incl. defaults, not just overrides
helm template mychart/ | kubectl apply --dry-run=client -f -    # validate rendered manifests against cluster API
helm uninstall myrelease --keep-history                          # uninstall but keep rollback history
helm search repo nginx --versions                                   # all available versions of a chart
```
