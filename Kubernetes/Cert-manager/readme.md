# Cert Manager (to manage SSL certs with Traefik in HA mode)

## Install Cert-manager

```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.0/cert-manager.yaml
```
## Let's Encrypt through Cloudflare DNS Challenge

Create a secret with Cloudflare API Token (all domains)

```
apiVersion: v1
kind: Secret
metadata:
  name: cloudflare-api-token-secret
type: Opaque
stringData:
  api-token: <API Token>
```

Then create the ClusterIssuer as follows.

```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: cert-manager-acme-issuer
  # Important: use the same namespace as Cert Manager deployment
  # Otherwise Cert Manager won't be able to find related elements
  namespace: cert-manager
spec:
  acme:
    # Email on which you'll receive notification for our certificates (expiration and such)
    email: pierre@crafteo.io
    # Name of the secret under which to store the secret key used by acme
    # This secret is managed by ClusterIssuer resource, you don't have to create it yourself
    privateKeySecretRef:
      name: cert-manager-acme-private-key
    # ACME server to use
    # Specify https://acme-v02.api.letsencrypt.org/directory for production
    # Staging server issues staging certificate which won't be trusted by most external parties but can be used for development purposes
    # server: https://acme-staging-v02.api.letsencrypt.org/directory
    server: https://acme-v02.api.letsencrypt.org/directory for production
    # Solvers define how to validate you're the owner of the domain for which to issue certificate
    # We use DNS-01 challenge with Cloudflare by providing related Cloudflare credentials (API Token) 
    solvers:
    - dns01:
        cloudflare:
          apiTokenSecretRef:
            name: cloudflare-api-token-secret
            key: api-token
```

## Promheteus monitoring

Deploy the following PodMonitor, after havign added the cert-manager namespace to the kube-prometheus-stack deployment

```
apiVersion: monitoring.coreos.com/v1
kind: PodMonitor
metadata:
  name: cert-manager
  namespace: cert-manager
  labels:
    app: cert-manager
    app.kubernetes.io/name: cert-manager
    app.kubernetes.io/instance: cert-manager
    app.kubernetes.io/component: "controller"
    release: kube-prometheus-stack
spec:
  jobLabel: app.kubernetes.io/name
  selector:
    matchLabels:
      app: cert-manager
      app.kubernetes.io/name: cert-manager
      app.kubernetes.io/instance: cert-manager
      app.kubernetes.io/component: "controller"
  podMetricsEndpoints:
    - port: http-metrics
      honorLabels: true
```
Search and add the cert-manager grafana dashboard

```
kubectl create configmap grafana-dashboard-cert-manager --from-file=grafana-cm.json
kubectl label configmap grafana-dashboard-cert-manager grafana_dashboard="1"
```
