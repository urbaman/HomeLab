# Heimadall Dashboard

Heimdall is an application dashboard which makes it very easy to organize you webpages and webapplications on a dashboard.
Let's see how to deploy it on kubernetes.

## Preparation

### Namespace

Create a dedicated namespace: heimdall

### DNS

Create a dedicated DNS entry heimdall.domain.com both on the external DNS (Cloudflare for me) and the internal DNS (pfSense for me), pointing the internal one on the Traefik loadbalancer IP.

### Persistent volume and claim

Create a volume, persistent volume and claim on longhorn, in the heimdall namespace, 1Gi of storage, ReadWriteMany as accessMode, Retain as ReclaimPolicy

Volume name: heimdall
Persistent Volume name: heimdall-pv
Persistent Volume Claim name: heimdall-pvc

### SSL cert

Create a certificate for heimdall.domain.com with cert-manager

## Deploy

Create the deployment, service and traefik ingress.

**Note:** I had to add the dnsConfig to properly resolve the app library URL in the yaml, and also change a `${var}` php variable to `{$var}` in a php file inside the pod to make it properly work.
{: .note}

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: heimdall        # < name of the deployment
  namespace: heimdall   # < namespace where place the deployment and pods
  labels:
    app: heimdall       # < label for tagging and reference
spec:
  replicas: 1           # < number of pods to deploy
  selector:
    matchLabels:
      app: heimdall
  strategy:
    rollingUpdate:
      maxSurge: 0       # < The number of pods that can be created above the desired amount of pods during an update
      maxUnavailable: 1 # < The number of pods that can be unavailable during the update process
    type: RollingUpdate # < New pods are added gradually, and old pods are terminated gradually
  template:
    metadata:
      labels:
        app: heimdall
    spec:
      volumes:
      - name: heimdall-volume  # < linkname of the volume for the pvc
        persistentVolumeClaim:
          claimName: heimdall-pvc # < pvc name we created in the previous yaml
      - name: heimdall-ssl
        secret:
          secretName: heimdall-domain # < the name ssl certificate, will be created in the ingress yaml
      containers:
      - image: ghcr.io/linuxserver/heimdall:latest # < the name of the docker image we will use
        name: heimdall                      # < name of container
        imagePullPolicy: Always             # < always use the latest image when creating container/pod
        env:                                # < the environment variables required (see container documentation)
        - name: PGID
          value: "100" # < group "user"
        - name: PUID
          value: "1041" # < user "docker"
        - name: TZ
          value: Europe/Rome
        ports:                              # < the ports required (see container documentation)
         - containerPort: 80
           name: http-80
           protocol: TCP
         - containerPort: 443
           name: https-443
           protocol: TCP
        volumeMounts:                       # < the volume mount in the container. Look at the relation volumelabel->pvc->pv
         - mountPath: /config               # < mount location in the container
           name: heimdall-volume              # < volumelabel configured earlier in the yaml file
           subPath: config                  # < subfolder in the nfs share to be mounted
      dnsConfig:                            # to properly resolve the app library URL
        options:
          - name: ndots
            value: "1"
---
apiVersion: v1
kind: Service
metadata:
  name: heimdall-service    # < name of the service
  namespace: heimdall       # < namespace where place the deployment and pods
spec:
  selector:
    app: heimdall           # < reference to the deployment (connects with this deployment)
  ports:
    - name: http-80
      protocol: TCP
      port: 80
    - name: https-443
      protocol: TCP
      port: 443
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: traefik-heimdall-basic-auth
  namespace: heimdall
spec:
  basicAuth:
    secret: traefik-heimdall-basic-auth
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: traefik-heimdall-https-redirect
  namespace: heimdall
spec:
  redirectScheme:
    scheme: https
    permanent: true
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: traefik-heimdall-security
  namespace: heimdall
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
  name: traefik-heimdall-transport
  namespace: heimdall
spec:
  serverName: traefik
  insecureSkipVerify: true
---
apiVersion: traefik.containo.us/v1alpha1
kind: TLSOption
metadata:
  name: traefik-heimdall-tlsoptions
  namespace: heimdall
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
  name: traefik-heimdall-basic-auth
  namespace: heimdall
data:
  users: |
    YWRtaW46JGFwcjEkR1VRTmlISlUkWkY2bkZuMHl2T3RLWnNWSG5ITmM4LgoK
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: traefik-heimdall-websecure
  namespace: heimdall
spec:
  entryPoints:
    - websecure
  routes:
    - kind: Rule
      match: Host(`heimdall.domain.com`)
      services:
        - name: heimdall-service
          port: https-443
          serversTransport: traefik-heimdall-transport
      middlewares:
        - name: traefik-heimdall-basic-auth
        - name: traefik-heimdall-security
  tls:
    secretName: heimdall-domain
    options:
      name: traefik-heimdall-tlsoptions
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: traefik-heimdall-web
  namespace: heimdall
spec:
  entryPoints:
    - web
  routes:
    - kind: Rule
      match: Host(`heimdall.urbaman.com`)
      services:
        - name: heimdall-service
          port: http-80
      middlewares:
        - name: traefik-heimdall-https-redirect
```
