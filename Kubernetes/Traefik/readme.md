# Traefik

## Create the glusterfs endpoints in the traefik namespace

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

## Create the path in the glusterfs volume (from a node in the glusterfs cluster)

```bash
cd /cluster/HDD5T
mkdir k8sstorage
cd k8sstorage
mkdir traefik-ssl
```

### Give the directory the right ownership

```bash
chown 65532:65532 /cluster/HDD5T/k8sstorage/traefik-ssl
```

## Create the pv and the pvc

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

## Get the helm chart

```bash
helm repo add traefik https://helm.traefik.io/traefik
helm repo update
```

## Modify the values to see the persistent volume claim

```bash
helm show values traefik/traefik > traefik.yaml
vi traefik.yaml
```

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
      certResolver: "cloudflare" # Or "pebble" if you so choose
      domains: []
      enabled: true
      options: ""
```

Enable the dashboard ingress route:

```bash
ingressRoute:
  dashboard:
    annotations: {}
    enabled: true
    labels: {}
```

Enable the ingressClass, set it to default:

```bash
ingressClass:
  enabled: true
  fallbackApiVersion: ""
  isDefaultClass: true
```

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

## Create SSL certs for your domains (Cloudflare DNS challenge for production OR pebble for test/dev)

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
