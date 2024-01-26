# Common commands (Linux)

## Export a given yaml key repetitions to a file

```bash
sed -rn 's/.*"(expr)": (.*)/\1 \2/p' prova.json
```

## Get pods in "Non-Ready" status"

```bash
kubectl get pods --field-selector="status.phase!=Succeeded,status.phase!=Running" -A
```
