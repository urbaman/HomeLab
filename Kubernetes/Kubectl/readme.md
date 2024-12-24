# Kubectl tips

Get all pods in a namespace with a given state

```bash
kubectl get pod -n <namespace> --field-selector status.phase=<State>
```

Delete all pods in a namespace with a given state

```bash
kubectl delete pod -n <namespace> --field-selector status.phase=<State>
```
