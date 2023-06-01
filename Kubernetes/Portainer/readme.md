# Portainer

To deploy Portainer, get the yaml manifest:

wget <https://raw.githubusercontent.com/portainer/k8s/master/deploy/manifests/portainer/portainer.yaml>

Add a persistent volume and a persistent volume claim to the manifest just after the service account (needs to be after the namespace creation, before the actual deploy creation), here we create it on glusterfs, you can do it on longhorn or any other storage solution, and reference the claim in the volume definition:

```yaml
---
# Source: portainer/templates/pvc.yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: glusterfs-cluster
  namespace: portainer
subsets:
  - addresses:
      - ip: 10.0.50.21
        hostname: truenas1
      - ip: 10.0.50.22
        hostname: truenas2
      - ip: 10.0.50.23
        hostname: truenas3
    ports:
      - port: 1
---
apiVersion: v1
kind: Service
metadata:
  name: glusterfs-cluster
  namespace: portainer
spec:
  ports:
  - port: 1
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: portainer-pv
  namespace: portainer
spec:
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 10Gi
  storageClassName: ""
  volumeMode: Filesystem
  glusterfs:
    endpoints: glusterfs-cluster
    path: HDD5T/k8storage/portainer
    readOnly: false
  claimRef:
    name: portainer-pvc
    namespace: portainer
  persistentVolumeReclaimPolicy: Retain
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: portainer-pvc
    namespace: portainer
spec:
    accessModes:
      - ReadWriteMany
    resources:
      requests:
        storage: 10Gi
    storageClassName: ""
    volumeName: portainer-pv
```

```yaml
      volumes:
        - name: "data"
          persistentVolumeClaim:
            claimName: portainer-pvc
```

Apply the manifest with kubectl

```bash
kubectl apply -f portainer.yaml
```

Go directly to the dashboard to create a user and login, or the instance will need to be restarted.

```bash
kubectl port-forward svc/portainer -n portainer 9000:9000
```

```bash
http://localhost:9000
```

Inside the dashboard, select the local environment, and go to to the Cluster/Setup page.

Here, define a "traefik" ingress class of type traefik, and enable "Enable features using the metrics API" (only if you installed the metrics server!). Save.

## Traefik exposure (admin portaineradmin)

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: portainer-urbaman
  namespace: portainer
spec:
  # Certificate will be valid for these domain names
  dnsNames:
  - portainer.urbaman.it
  # Reference our issuer
  # As it's a ClusterIssuer, it can be in a different namespace
  issuerRef:
    kind: ClusterIssuer
    name: cert-manager-acme-issuer
  # Secret that will be created with our certificate and private keys
  secretName: portainer-urbaman
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: portainer-dashboard-basic-auth
  namespace: portainer
spec:
  basicAuth:
    secret: portainer-dashboard-basic-auth
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: portainer-dashboard-https-redirect
  namespace: portainer
spec:
  redirectScheme:
    scheme: https
    permanent: true
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: portainer-dashboard-security
  namespace: portainer
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
  name: portainer-dashboard-transport
  namespace: portainer
spec:
  serverName: portainer
  insecureSkipVerify: true
---
apiVersion: traefik.containo.us/v1alpha1
kind: TLSOption
metadata:
  name: portainer-dashboard-tlsoptions
  namespace: portainer
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
  name: portainer-dashboard-basic-auth
  namespace: portainer
data:
  users: |
    YWRtaW46JGFwcjEkcjBTZGM5alAkVDJISDZDNmkvOXhuUTdJS3NobjAwMAoK
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: portainer-dashboard-websecure
  namespace: portainer
spec:
  entryPoints:
    - websecure
  routes:
    - kind: Rule
      match: Host(`portainer.urbaman.it`)
      services:
      - name: portainer
        port: 9443
        serversTransport: portainer-dashboard-transport
      middlewares:
#        - name: portainer-dashboard-basic-auth
        - name: portainer-dashboard-security
  tls:
    secretName: portainer-urbaman
    options:
      name: portainer-dashboard-tlsoptions
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: portainer-dashboard-web
  namespace: portainer
spec:
  entryPoints:
    - web
  routes:
    - kind: Rule
      match: Host(`portainer.urbaman.it`)
      services:
      - name: portainer
        port: 9443
      middlewares:
        - name: portainer-dashboard-https-redirect
```
