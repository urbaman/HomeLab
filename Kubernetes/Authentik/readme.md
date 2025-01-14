# Deploy Authentik

## Prerequisites

You better have a dedicated postgres db with dedicated user and a redis server.

Create a random secret_key:

```bash
openssl rand 60 | base64 -w 0
```

## Helm deploy

```bash
helm repo add authentik https://charts.goauthentik.io
helm repo update
helm show values authentik/authentik > authentik-values.yaml
vi authentik-values.yaml
helm upgrade -i authentik -n authentik --create-namespace authentik/authentik -f authentik-values.yaml
kubectl apply -f ig-authentik.yaml
```
