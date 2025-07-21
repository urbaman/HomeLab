# Valkey deployment (Redis alternative)

```bash
helm show values oci://registry-1.docker.io/bitnamicharts/valkey > valkey-values.yaml
vi valkey-values.yaml
```

Set architecture, resources, persistence, metrics (enable serviceMonitor and prometheusRules)

```bash
helm upgrade -i valkey -n valkey --create-namespace oci://registry-1.docker.io/bitnamicharts/valkey -f valkey-values.yaml
```

Get the password

```bash
kubectl get secret --namespace valkey valkey -o jsonpath="{.data.valkey-password}" | base64 -d
```
