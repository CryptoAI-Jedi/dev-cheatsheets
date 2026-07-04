# Kubernetes Cheatsheet

> kubectl for daily driving: pods, deployments, services, and the debugging loop (describe → logs → exec). Almost every answer starts with `kubectl describe` and its Events section.

---

## Table of Contents
- [Cluster Info](#cluster-info)
- [Namespaces](#namespaces)
- [Pods](#pods)
- [Deployments](#deployments)
- [Services](#services)
- [ConfigMaps & Secrets](#configmaps--secrets)
- [Persistent Volumes](#persistent-volumes)
- [Resource Inspection](#resource-inspection)
- [Resource Management](#resource-management)
- [Debugging](#debugging)
- [Common Manifest Snippets](#common-manifest-snippets)
- [Tips & Gotchas](#tips--gotchas)

---

## CLUSTER INFO

```text
kubectl version                                 → Client & server version (--short removed in modern kubectl)
kubectl cluster-info                            → Cluster endpoint info
kubectl config get-contexts                     → List available contexts
kubectl config current-context                  → Show active context
kubectl config use-context context-name         → Switch context
kubectl get nodes                               → List nodes
kubectl get nodes -o wide                       → Nodes with IPs & OS info
kubectl describe node node-name                 → Detailed node info
kubectl top nodes                               → CPU/mem usage per node (needs metrics-server)
```

---

## NAMESPACES

```text
kubectl get namespaces                          → List all namespaces
kubectl create namespace my-ns                  → Create namespace
kubectl delete namespace my-ns                  → Delete namespace (and EVERYTHING in it)
kubectl config set-context --current --namespace=my-ns   → Set default namespace
```

---

## PODS

```text
kubectl get pods                                → List pods (current namespace)
kubectl get pods -A                             → List pods (all namespaces)
kubectl get pods -o wide                        → Pods with node & IP info
kubectl describe pod pod-name                   → Full pod details & events
kubectl logs pod-name                           → Pod logs
kubectl logs pod-name -c container-name         → Logs from specific container
kubectl logs -f pod-name                        → Follow (tail) pod logs
kubectl logs --previous pod-name                → Logs from crashed container
kubectl exec -it pod-name -- bash               → Shell into pod
kubectl exec -it pod-name -c container -- bash  → Shell into specific container
kubectl delete pod pod-name                     → Delete pod (restarts if managed)
kubectl delete pod pod-name --force             → Force delete stuck pod (last resort)
kubectl get pod pod-name -o yaml                → Full pod spec as YAML
```

---

## DEPLOYMENTS

```text
kubectl get deployments                         → List deployments
kubectl describe deployment deploy-name         → Deployment details
kubectl create deployment myapp --image=nginx   → Quick create deployment
kubectl apply -f deployment.yaml                → Apply manifest
kubectl delete -f deployment.yaml               → Delete from manifest
kubectl scale deployment myapp --replicas=3     → Scale replicas
kubectl rollout status deployment myapp         → Watch rollout progress
kubectl rollout history deployment myapp        → Revision history
kubectl rollout undo deployment myapp           → Rollback to previous revision
kubectl rollout undo deployment myapp --to-revision=2   → Rollback to specific rev
kubectl rollout restart deployment myapp        → Rolling restart (fresh pods, same spec)
kubectl set image deployment/myapp app=nginx:1.25   → Update container image
```

---

## SERVICES

```text
kubectl get services                            → List services
kubectl describe service svc-name               → Service details
kubectl expose deployment myapp --port=80 --type=ClusterIP     → Expose internally
kubectl expose deployment myapp --port=80 --type=NodePort      → Expose on node port
kubectl expose deployment myapp --port=80 --type=LoadBalancer  → Cloud LB
kubectl delete service svc-name                 → Delete service
kubectl port-forward pod/pod-name 8080:80       → Forward local port to pod
kubectl port-forward svc/svc-name 8080:80       → Forward local port to service
```

---

## CONFIGMAPS & SECRETS

```text
kubectl get configmaps                          → List configmaps
kubectl describe configmap cm-name              → ConfigMap details
kubectl create configmap my-cm --from-file=config.txt
kubectl create configmap my-cm --from-literal=KEY=VALUE
```

```text
kubectl get secrets                             → List secrets
kubectl describe secret secret-name             → Secret details (values hidden)
kubectl create secret generic my-secret --from-literal=password=abc123
kubectl get secret my-secret -o jsonpath='{.data.password}' | base64 -d   → Decode
```

---

## PERSISTENT VOLUMES

```text
kubectl get pv                                  → List persistent volumes
kubectl get pvc                                 → List persistent volume claims
kubectl describe pvc pvc-name                   → PVC details
kubectl delete pvc pvc-name                     → Delete PVC
```

---

## RESOURCE INSPECTION

```text
kubectl get all                                 → All resources in namespace
kubectl get all -A                              → All resources cluster-wide
kubectl get events --sort-by=.metadata.creationTimestamp   → Sorted events
kubectl get events -A                           → Events across all namespaces
kubectl api-resources                           → List all resource types
kubectl explain pod.spec.containers             → Built-in docs for ANY field
```

---

## RESOURCE MANAGEMENT

```text
kubectl apply -f file.yaml                      → Create or update resource
kubectl delete -f file.yaml                     → Delete from manifest
kubectl edit deployment myapp                   → Live edit resource
kubectl patch deployment myapp -p '{"spec":{"replicas":2}}'   → Patch field
kubectl label pod pod-name env=prod             → Add label
kubectl annotate pod pod-name desc="my pod"     → Add annotation
kubectl cordon node-name                        → Mark node unschedulable
kubectl uncordon node-name                      → Re-enable scheduling
kubectl drain node-name --ignore-daemonsets     → Evict pods from node (maintenance)
```

---

## DEBUGGING

```text
kubectl describe pod pod-name                   → Check Events section FIRST
kubectl get pod pod-name -o yaml                → Full spec (check status.conditions)
kubectl logs pod-name --previous                → Logs before crash
kubectl exec -it pod-name -- sh                 → Shell in (sh — most images lack bash)
kubectl run debug --rm -it --image=busybox -- sh   → Temp debug pod
kubectl get endpoints svc-name                  → Check if service has backends
kubectl auth can-i create pods                  → Check RBAC permissions
```

---

## COMMON MANIFEST SNIPPETS

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: nginx:1.25        # Pin versions — :latest drifts (see Gotchas)
          ports:
            - containerPort: 80
```

### Service (ClusterIP)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-svc
spec:
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 80
```

---

## TIPS & GOTCHAS

- **The answer is in `describe`'s Events** — ImagePullBackOff, FailedScheduling, OOMKilled: the Events section at the bottom of `kubectl describe pod` names the cause before you ever read logs.
- **CrashLoopBackOff → `logs --previous`** — the current container hasn't logged yet; the crash reason is in the *previous* container's output.
- **Check your namespace before panicking** — "my pods are gone" is usually "I'm in the wrong namespace." `kubectl config set-context --current --namespace=` or `-n` per command; `-A` when hunting.
- **Selector ↔ label mismatch = service with no backends** — `kubectl get endpoints svc-name` returning `<none>` means the Service selector doesn't match any pod labels. It's the #1 "service doesn't work" cause.
- **Secrets are base64, not encryption** — anyone with `get secret` reads the values in one pipe. RBAC and encryption-at-rest are the protections, not the encoding.
- **Pin image tags in manifests** — `:latest` plus `imagePullPolicy` ambiguity means different nodes can run different code. Tag per release; `rollout restart` for redeploys.
- **`--force` deleting pods can double-run workloads** — force delete removes the API object, not necessarily the container on the node. Fine for stateless; dangerous for anything holding a lock.
- **`kubectl explain` beats searching docs** — `kubectl explain deployment.spec.strategy` gives versioned, cluster-accurate field docs offline.

---
*Last Updated: 2026-07*
