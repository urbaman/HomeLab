# Collabora Online installation

Create a secret with collabora online username and password, install the helm chart with the provided collabora-values.yaml values file, then apply the traefik ig-collabora.yaml

```bash
kubectl create secret generic -n collabora-online collabora-auth \
    --from-literal=username='USERNAME' \
    --from-literal=password='PASSWORD'
helm repo add collabora https://collaboraonline.github.io/online/
helm repo update
helm show values collabora/collabora-online > collabora-values.yaml
```

Set the values you want or need (probaly resources requests and limits), see example.

```bash
helm upgrade -i --create-namespace --namespace collabora-online collabora-online collabora/collabora-online -f collabora-values.yaml
kubectl apply -f ig-collabora.yaml
```
