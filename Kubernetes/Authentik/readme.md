# Deploy Authentik

## Prerequisites

You better have a dedicated postgres db with dedicated user and a redis server.

Create a random secret_key:

```bash
openssl rand 60 | base64 -w 0
```

Add the

```yaml
      containers:
      - args:
        - --providers.kubernetescrd.allowCrossNamespace=true
```

To the traefik deployment (or directly in the helm chart)

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

## Proxy provider

**Provider**
- Name: Home forward auth
- Authorization flow: default-provider-authorization-explicit-consent
- Type: Forward auth (domain level)
- Authentication URL: https://authentik.domain.com
- Cookie domain: domain.com

**Application**
- Name: Home
- Provider: Home forward auth
- Launch URL: blank://blank (this hides the application in the library page, “my applications”)

Navigate to `Applications > Outposts`. Edit the authentik Embedded Outpost and make sure that the entry for the application Home in the field Applications is selected.

Then add the `middleware-authentik.yaml` to the namespace of the websecure ingressroute you want to protect, and change the websecure ingressroute as in `ig-authentik-forwardauth.yaml`.
