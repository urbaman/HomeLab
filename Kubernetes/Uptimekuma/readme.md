# Uptime Kuma for service uptime monitoring

## Install Uptime Kuma through helm

Once Helm has been set up correctly, add the repo as follows:


```bash
helm repo add uptime-kuma https://dirsigler.github.io/uptime-kuma-helm
```

If you had already added this repo earlier, run helm repo update to retrieve the latest versions of the packages. You can then run helm search repo uptime-kuma to see the charts.

Get the values and change the version to latest:

```yaml
image:
  pullPolicy: IfNotPresent
  repository: louislam/uptime-kuma
  tag: latest
```

To install the uptime-kuma chart:

```bash
helm upgrade my-uptime-kuma uptime-kuma/uptime-kuma --install --namespace monitoring --values ukuma.yaml
```

## Expose through IngressRoute

Create and install the ssl cert and ingressroute

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: uptimekuma-urbaman
  namespace: monitoring
spec:
  # Certificate will be valid for these domain names
  dnsNames:
  - uptimekuma.urbaman.it
  # Reference our issuer
  # As it's a ClusterIssuer, it can be in a different namespace
  issuerRef:
    kind: ClusterIssuer
    name: cert-manager-acme-issuer
  # Secret that will be created with our certificate and private keys
  secretName: uptimekuma-ssl-cert
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: uptimekuma-basic-auth
  namespace: monitoring
spec:
  basicAuth:
    secret: uptimekuma-basic-auth
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: uptimekuma-https-redirect
  namespace: monitoring
spec:
  redirectScheme:
    scheme: https
    permanent: true
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: uptimekuma-security
  namespace: monitoring
spec:
  headers:
    frameDeny: true
    sslRedirect: true
    browserXssFilter: true
    contentTypeNosniff: true
    stsIncludeSubdomains: true
    stsPreload: true
    stsSeconds: 31536000
---
apiVersion: traefik.containo.us/v1alpha1
kind: TLSOption
metadata:
  name: uptimekuma-tlsoptions
  namespace: monitoring
spec:
  minVersion: VersionTLS12
  cipherSuites:
    - TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
    - TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
    - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
    - TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305
    - TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
    - TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305
    - TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305
    - TLS_AES_256_GCM_SHA384
    - TLS_AES_128_GCM_SHA256
    - TLS_CHACHA20_POLY1305_SHA256
    - TLS_FALLBACK_SCSV
  curvePreferences:
    - CurveP521
    - CurveP384
  sniStrict: false
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: uptimekuma-websecure
  namespace: monitoring
spec:
  entryPoints:
    - websecure
  routes:
  - match: Host(`uptimekuma.urbaman.it`)
    kind: Rule
    services:
    - name: uptime-kuma
      port: 3001
    middlewares:
    - name: uptimekuma-security
  tls:
    secretName: uptimekuma-ssl-cert
    options:
      name: uptimekuma-tlsoptions
      namespace: monitoring
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: uptimekuma-web
  namespace: monitoring
spec:
  entryPoints:
    - web
  routes:
  - match: Host(`uptimekuma.urbaman.it`)
    kind: Rule
    services:
    - name: uptime-kuma
      port: 3001
    middlewares:
      - name: uptimekuma-https-redirect
```

## Create the Prometheus Service Monitor

Create a secret with the basic auth base64 username and password, then the service monitor

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: uptime-kuma-basic-auth
  namespace: monitoring
data:
  username: <base64 username>
  password: <base64 password>
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: uptimekuma-sm
  namespace: monitoring
  labels:
    release: kube-prometheus-stack
spec:
  jobLabel: uptimekuma
  namespaceSelector:
    matchNames:
    - monitoring
  selector:
    matchLabels:
      app.kubernetes.io/instance: uptime-kuma
      app.kubernetes.io/name: uptime-kuma
  endpoints:
  - port: http
    interval: 15s
    path: '/metrics'
    basicAuth: # Only needed if authentication is enabled (default)
      username:
        name: uptime-kuma-basic-auth
        key: username
      password:
        name: uptime-kuma-basic-auth
        key: password
```

Then, import and define the grafana dashboard.