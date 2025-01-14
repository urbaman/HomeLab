# Deploy Authentik

## Prerequisites

You better have a dedicated postgres db with dedicated user and a redis server.

Create a random secret_key:

```bash
openssl rand 60 | base64 -w 0
```

## Helm deploy

In the authentik-values.yaml file, set the secret-key value, metrics and servicemonitor to true, servicemonitor labels to release: kube-prometheus-stack, geoip.enabled to true, fill the maxmind apikey values, and the email specifics for the notifications, and/or whatever you need

```bash
helm repo add authentik https://charts.goauthentik.io
helm repo update
helm show values authentik/authentik > authentik-values.yaml
vi authentik-values.yaml
helm upgrade -i authentik -n authentik --create-namespace authentik/authentik -f authentik-values.yaml
kubectl apply -f ig-authentik.yaml
```
