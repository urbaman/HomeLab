# Proxmox Backup Monitoring with prometheus and pushgateway

## Install prometheus pushgateway on kubernetes

```bash
helm show values prometheus-community/prometheus-pushgateway > push.yaml
```

Define the namespace to monitoring, the Service Monitor to true and add the release=kube-prometheus-stack to the Service Monitor.

```bash
helm install prometheus-pushgateway prometheus-community/prometheus-pushgateway --values push.yaml -n monitoring
```

Then, create the ingressroute

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: pushgateway-urbaman
  namespace: monitoring
spec:
  # Certificate will be valid for these domain names
  dnsNames:
  - pushgateway.urbaman.it
  # Reference our issuer
  # As it's a ClusterIssuer, it can be in a different namespace
  issuerRef:
    kind: ClusterIssuer
    name: cert-manager-acme-issuer
  # Secret that will be created with our certificate and private keys
  secretName: pushgateway-urbaman
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: monitoring-pushgateway-basic-auth
  namespace: monitoring
spec:
  basicAuth:
    secret: monitoring-pushgateway-basic-auth
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: monitoring-pushgateway-https-redirect
  namespace: monitoring
spec:
  redirectScheme:
    scheme: https
    permanent: true
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: monitoring-pushgateway-security
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
apiVersion: traefik.io/v1alpha1
kind: ServersTransport
metadata:
  name: monitoring-pushgateway-transport
  namespace: monitoring
spec:
  serverName: pushgateway
  insecureSkipVerify: true
---
apiVersion: traefik.containo.us/v1alpha1
kind: TLSOption
metadata:
  name: monitoring-pushgateway-tlsoptions
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
apiVersion: v1
kind: Secret
metadata:
  name: monitoring-pushgateway-basic-auth
  namespace: monitoring
data:
  users: |
    YWRtaW46JGFwcjEkSVBqUU96bmYkWkhDSWp2UmdTNFd4bXBUZnVVakhMMQoK
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: monitoring-pushgateway-websecure
  namespace: monitoring
spec:
  entryPoints:
    - websecure
  routes:
    - kind: Rule
      match: Host(`pushgateway.urbaman.it`)
      services:
      - name: prometheus-pushgateway
        port: 9091
        serversTransport: monitoring-pushgateway-transport
      middlewares:
        - name: monitoring-pushgateway-security
  tls:
    secretName: pushgateway-urbaman
    options:
      name: monitoring-pushgateway-tlsoptions
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: monitoring-pushgateway-web
  namespace: monitoring
spec:
  entryPoints:
    - web
  routes:
    - kind: Rule
      match: Host(`pushgateway.urbaman.it`)
      services:
      - name: prometheus-pushgateway
        port: 9091
      middlewares:
        - name: monitoring-pushgateway-https-redirect
```

## Proxmox Backup Server exporter

Install the Proxmox Backup Server Exporter from https://github.com/rare-magma/pbs-exporter

**Note:** you'll probably need to tweak it a little bit to make it properly source the conf file.
{: .note}

## Add grafana Dashboard

Get it here: https://grafana.com/grafana/dashboards/18740-proxmox-backup-server/