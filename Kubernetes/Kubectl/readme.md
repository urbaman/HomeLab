# Kubectl tips

Get all pods in a namespace with a given state

```bash
kubectl get pod -n <namespace> --field-selector status.phase=<State>
```

Delete all pods in a namespace with a given state

```bash
kubectl delete pod -n <namespace> --field-selector status.phase=<State>
```

Get pods not ready in a namespace

```bash
kubectl get pods -n <namespace> | grep "0/"
```

Add a counter to the previous commands

```bash
<command> | wc -l
```
