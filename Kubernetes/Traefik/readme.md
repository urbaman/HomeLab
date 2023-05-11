# Traefik

## Gluster for storage (use Longhorn instead, not needed for HA through cert-manager)

### Create the glusterfs endpoints in the traefik namespace

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: glusterfs-cluster
  namespace: traefik
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
  namespace: traefik
spec:
  ports:
  - port: 1
```

### Create the path in the glusterfs volume (from a node in the glusterfs cluster)

```bash
cd /cluster/HDD5T
mkdir k8sstorage
cd k8sstorage
mkdir traefik-ssl
```

#### Give the directory the right ownership

```bash
chown 65532:65532 /cluster/HDD5T/k8sstorage/traefik-ssl
```

### Create the pv and the pvc

```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: traefik-ssl-volume
spec:
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 128Mi
  storageClassName: ""
  volumeMode: Filesystem
  glusterfs:
    endpoints: glusterfs-cluster
    path: HDD5T/k8storage/traefik-ssl
    readOnly: false
  claimRef:
    name: traefik-ssl-claim
    namespace: traefik
  persistentVolumeReclaimPolicy: Retain
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: traefik-ssl-claim
    namespace: traefik
spec:
    accessModes:
      - ReadWriteMany
    resources:
      requests:
        storage: 128Mi
    storageClassName: ""
    volumeName: traefik-ssl-volume
```

## Longhorn (not needed for HA through cert-manager)

### Create the pv and the pvc

```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: traefik-ssl-pv
spec:
  capacity:
    storage: 2Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: longhorn
  csi:
    driver: driver.longhorn.io
    fsType: ext4
    volumeAttributes:
      numberOfReplicas: '3'
      staleReplicaTimeout: '2880'
    volumeHandle: traefik-ssl-volume
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: traefik-ssl-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
  volumeName: traefik-ssl-pv
  storageClassName: longhorn
```

## Get the helm chart

```bash
helm repo add traefik https://helm.traefik.io/traefik
helm repo update
```

```bash
helm show values traefik/traefik > traefik.yaml
vi traefik.yaml
```

## Modify the values to see the persistent volume claim (Only for no HA with Treafik's cert managing)

Go to the persistence section, turn it to true, uncomment the existingClaim line and set it to the claim above.

```bash
persistence:
  accessMode: ReadWriteMany
  annotations: {}
  enabled: true
  existingClaim: traefik-ssl-claim
  name: ssl-certs
  path: /ssl-certs
  size: 128Mi
```

## Other changes and settings

Go to the ports section, and set to true both the metrics and the traefik (dashboard) expose settings. Also add a redirect from web to websecure and the default tls resolver:

```bash
ports:
  metrics:
    expose: true
    exposedPort: 9100
    port: 9100
    protocol: TCP
  traefik:
    expose: true
    exposedPort: 9000
    port: 9000
    protocol: TCP
  web:
    expose: true
    exposedPort: 80
    port: 8000
    protocol: TCP
    redirectTo: websecure
  websecure:
    expose: true
    exposedPort: 443
    port: 8443
    protocol: TCP
    tls:
      certResolver: "cloudflare" # Only for non HA, set to "pebble" if you so choose
      domains: []
      enabled: true
      options: ""
      secret: secret-name # add this for HA settings, with the secret containing the cert-manager certificate for the traefik host to reach
```

Enable the dashboard ingress route:

```bash
ingressRoute:
  dashboard:
    annotations: {}
    enabled: true
    labels: {}
    tls:
      secret: secret-name # add this for HA settings, with the secret containing the cert-manager certificate for the traefik host to reach
```

Enable the ingressClass, set it to default:

```bash
ingressClass:
  enabled: true
  fallbackApiVersion: ""
  isDefaultClass: true
```

## SSL certs permisisons check (Only for no HA with Treafik's cert managing)

Finally go to the deployment section, and add the following initContainer to properly manage certificates permissions in case of errors (it will probably not run and break traefik's start process):

```bash
deployment:
  additionalContainers: []
  additionalVolumes: []
  annotations: {}
  enabled: true
  imagePullSecrets: []
  initContainers:
    - name: volume-permissions
      image: busybox:1.31.1
      command: ["sh", "-c", "chmod -Rv 600 /ssl-certs/*"]
      volumeMounts:
        - name: ssl-certs
          mountPath: /ssl-certs
```

## Create SSL certs for your domains (Cloudflare DNS challenge for production OR pebble for test/dev) - (Only for no HA with Treafik's cert managing)

Create a secret with your own Cloudlfare email and API Key:

```bash
apiVersion: v1
kind: Secret
metadata:
  name: cloudflare-dnschallenge-credentials
  namespace: traefik
type: Opaque
stringData:
  email: yourmail@example.com
  apiKey: yourglobalapikey
```

Also in the traefik vales yaml file, add the following additionalArguments (choose only one resolver), the env settings and the volume settings (volume only for pebble certResolver)

```bash
additionalArguments:
# DNS Challenge
# Cloudflare
  - --certificatesresolvers.cloudflare.acme.dnschallenge.provider=cloudflare
  - --certificatesresolvers.cloudflare.acme.email=yourmail@example.com
  - --certificatesresolvers.cloudflare.acme.dnschallenge.resolvers=1.1.1.1
  - --certificatesresolvers.cloudflare.acme.storage=/ssl-certs/acme-cloudflare.json
```

```bash
env:
# Only for Pebble letsencrypt challenge, comment or delete for others
- name: LEGO_CA_CERTIFICATES
  value: "/certs/root-cert.pem"
# DNS Challenge Credentials
# Cloudflare:
- name: CF_API_EMAIL
  valueFrom:
    secretKeyRef:
      key: email
      name: cloudflare-dnschallenge-credentials
- name: CF_API_KEY
  valueFrom:
    secretKeyRef:
      key: apiKey
      name: cloudflare-dnschallenge-credentials
```

## Install

```bash
helm install traefik traefik/traefik --values traefik.yaml -n traefik
```

## Check the dashboard

Go to a machine with a browser and kubectl installed and properly configured, open the terminal

```bash
kubectl -n traefik port-forward traefik-667459fcbf-h7cqf 9000:9000
```

Now point the browser to <http://localhost:9000/dashboard/> (remember the final slash)

## Examples

### Proxy an internal service (https redirect, basic auth)

Generate a username/password base64 hash to use in the basicauth secret:

```bash
htpasswd -nb username password | base64
dXNlcm5hbWU6JGFwcjEkLmtMR05oTzAkcnZVZTZBY0k3R2dyRHlFbkI0d2J2LwoK
```

Customize and apply the following yaml file (this is for the k8s dashboard previously exposed, can point to any exposed service):

```bash
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: k8dashboard-basic-auth
  namespace: kubernetes-dashboard
spec:
  basicAuth:
    secret: k8dashboard-basic-auth
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: k8dashboard-https-redirect
  namespace: kubernetes-dashboard
spec:
  redirectScheme:
    scheme: https
    permanent: true
---
apiVersion: v1
kind: Secret
metadata:
  name: k8dashboard-basic-auth
  namespace: kubernetes-dashboard
data:
  users: |
    dXNlcm5hbWU6JGFwcjEkLmtMR05oTzAkcnZVZTZBY0k3R2dyRHlFbkI0d2J2LwoK
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: k8dashboard-websecure
  namespace: kubernetes-dashboard
spec:
  entryPoints:
    - websecure
  routes:
  - match: Host(`k8dashboard.example.com`)
    kind: Rule
    services:
    - name: kubernetes-dashboard
      port: 80
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: k8dashboard-web
  namespace: kubernetes-dashboard
spec:
  entryPoints:
    - web
  routes:
  - match: Host(`k8dashboard.example.com`)
    kind: Rule
    services:
    - name: kubernetes-dashboard
      port: 80
    middlewares:
      - name: k8dashboard-https-redirect
```

### Proxy an external service

You first need to define endpoitns and service

```bash
kind: Endpoints
apiVersion: v1
metadata:
  name: truenas1
  namespace: truenas
subsets:
  - addresses:
    - ip: 10.0.50.21
    ports:
    - port: 80
      name: truenas1
---
apiVersion: v1
kind: Service
metadata:
  name: truenas1
  namespace: truenas
spec:
  ports:
     - protocol: TCP
       port: 80
       targetPort: 80
       name: truenas1
       nodePort: 0
```

Then you can set an ingressroute to the service as an internal service.

## Security updates to get A+ on SSL labs checks

Add a security middleware and a TLSoptions object, add them to the IngressRoute

```
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: security
  namespace: dev
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
  name: tlsoptions
  namespace: dev
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
  name: whoami
  namespace: dev
spec:
  entryPoints:
    - websecure
  routes:
  - kind: Rule
    match: Host(`whoami.20.115.56.189.nip.io`)
    services:
    - name: whoami
      port: 80
    middlewares:
    - name: security
  tls:
    certResolver: le
    options:
      name: tlsoptions
      namespace: dev
```

## High availability (with cert-manager)

To reach high availability we need an external tool for SSL cert management (cert-manager is the one we choose)

In the traefik.yaml values file, add/change theese values to reach high availability, do not execute the changes for TLS management, as indicated:

```
# We want an highly available Traefik
# Use a Deployment with nodeAffinity spreading our Pods across nodes
# We could also use a DaemonSet to have an instance par node
deployment:
  replicas: 3

# Using affinity from Traefik default values example
# This pod anti-affinity forces the scheduler to put traefik pods
# on nodes where no other traefik pods are scheduled.
# It should be used when hostNetwork: true to prevent port conflicts
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - topologyKey: failure-domain.beta.kubernetes.io/zone
      labelSelector:
        matchExpressions:
        - key: app
          operator: In
          values: 
          - traefik

# Automatically redirect http to https
# Not required but handy
ports:
  web:
    redirectTo: websecure
```

Then, create the certificates with cert-manager and add them to the IngressRoute:

```
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: whoami
spec:
  entryPoints:
    - websecure
  routes:
    - match: Host(`whoami.example.com`)
      kind: Rule
      services:
        - name: whoami
          port: 80
  tls:
    secretName: whoami
```
