# Kubernetes (kubectl) Cheat Sheet

## Context & config
```bash
kubectl config get-contexts
kubectl config current-context
kubectl config use-context my-cluster
kubectl config set-context --current --namespace=my-ns
kubectl cluster-info
kubectl version
```

## Getting resources
```bash
kubectl get pods                              # pods in current namespace
kubectl get pods -A                            # pods in all namespaces
kubectl get pods -o wide                        # extra columns (node, IP)
kubectl get pods -w                              # watch for changes
kubectl get all                                    # most resources in namespace
kubectl get deploy,svc,ing                          # specific types
kubectl get pod mypod -o yaml                         # full manifest
kubectl get pod mypod -o json | jq .spec.containers
kubectl get pods -l app=web                             # filter by label
kubectl get pods --field-selector=status.phase=Running
```

## Describing & debugging
```bash
kubectl describe pod mypod                # events, conditions, config
kubectl logs mypod                          # container logs
kubectl logs -f mypod                        # follow
kubectl logs mypod -c container-name           # specific container in multi-container pod
kubectl logs --previous mypod                    # logs from previous crashed instance
kubectl exec -it mypod -- bash                     # shell into pod
kubectl exec -it mypod -c container-name -- sh
kubectl top pod                                       # resource usage (needs metrics-server)
kubectl top node
kubectl get events --sort-by=.lastTimestamp
```

## Creating & applying
```bash
kubectl apply -f deployment.yaml
kubectl apply -f ./manifests/                      # apply a directory
kubectl delete -f deployment.yaml
kubectl create deployment web --image=nginx
kubectl run tmp --image=busybox --rm -it -- sh      # quick throwaway debug pod
kubectl expose deployment web --port=80 --target-port=8080
kubectl scale deployment web --replicas=5
kubectl rollout restart deployment web
kubectl rollout status deployment web
kubectl rollout undo deployment web
kubectl rollout history deployment web
```

## Editing & patching
```bash
kubectl edit deployment web                          # live edit in $EDITOR
kubectl set image deployment/web web=nginx:1.25
kubectl patch deployment web -p '{"spec":{"replicas":3}}'
kubectl label pod mypod env=prod
kubectl annotate pod mypod note="deploy 2026-08-02"
```

## Namespaces & context
```bash
kubectl get ns
kubectl create ns staging
kubectl delete ns staging
kubectl config set-context --current --namespace=staging
```

## Port-forwarding & proxy
```bash
kubectl port-forward pod/mypod 8080:80
kubectl port-forward svc/web 8080:80
kubectl proxy                                # local proxy to API server
```

## ConfigMaps & Secrets
```bash
kubectl create configmap myconf --from-file=config.yaml
kubectl create secret generic mysecret --from-literal=password=hunter2
kubectl get secret mysecret -o jsonpath='{.data.password}' | base64 -d
kubectl create secret docker-registry regcred --docker-server=... --docker-username=... --docker-password=...
```

## Useful one-liners
```bash
kubectl get pods --field-selector=status.phase!=Running          # non-running pods
kubectl get pods -A -o wide | grep -v Running                     # pods not running, all ns
kubectl delete pod mypod --grace-period=0 --force                  # force delete stuck pod
kubectl get pods -o=jsonpath='{.items[*].metadata.name}'             # just pod names
kubectl get nodes -o wide
kubectl cordon node1 && kubectl drain node1 --ignore-daemonsets      # prep node for maintenance
kubectl uncordon node1
kubectl explain pod.spec.containers                                   # inline API docs
kubectl diff -f deployment.yaml                                         # preview changes before apply
kubectl get pod -o custom-columns=NAME:.metadata.name,STATUS:.status.phase
watch kubectl get pods                                                    # live-refresh pod list
kubectl api-resources                                                       # list all resource types
```
