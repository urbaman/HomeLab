# Longhorn kubernetes storage solution

## Environment check

Download and run the environment check

```bash
sudo apt install kubectl jq nfs-common -y
curl -sSfL https://raw.githubusercontent.com/longhorn/longhorn/v1.5.1/scripts/environment_check.sh | bash
```

## Installation

```bash
wget https://raw.githubusercontent.com/longhorn/longhorn/v1.5.1/deploy/longhorn.yaml
```

Edit the settings adding needed lines to the longhorn-default-setting ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: longhorn-default-setting
  namespace: longhorn-system
  labels:
    app.kubernetes.io/name: longhorn
    app.kubernetes.io/instance: longhorn
    app.kubernetes.io/version: v1.5.1
data:
  default-setting.yaml: |-
    default-data-path: /longhorn/storage/ # this is the path where the primary storage is mounted
    storage-network: default/ipvlan-conf # this is the name of the multus network-attachment-definition
    default-replica-count: 6
```

Eventually set the number of replicas in the StorageClass, then install.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: longhorn-storageclass
  namespace: longhorn-system
  labels:
    app.kubernetes.io/name: longhorn
    app.kubernetes.io/instance: longhorn
    app.kubernetes.io/version: v1.5.1
data:
  storageclass.yaml: |
    kind: StorageClass
    apiVersion: storage.k8s.io/v1
    metadata:
      name: longhorn
      annotations:
        storageclass.kubernetes.io/is-default-class: "true"
    provisioner: driver.longhorn.io
    allowVolumeExpansion: true
    reclaimPolicy: "Delete"
    volumeBindingMode: Immediate
    parameters:
      numberOfReplicas: "3"
      staleReplicaTimeout: "30"
      fromBackup: ""
      fsType: "ext4"
      dataLocality: "disabled"
```


```bash
kubectl apply -f longhorn.yaml
```

## Access the gui

From your client with kubectl installed:

```bash
kubectl port-forward -n longhorn-system service/longhorn-frontend :80
```

Go to <http://IP-OF-A-K8S-CONTROL-PANEL:PORT-SHOWN-IN-OUTPUT>

Check that all nodes are running and scheduling, and that the storage space is as expected.

## Check

### With PV

Create a volume from the Longhorn UI named test-volume, then create the PV (beware to use the volume name in volumeHandle), PVC and POD

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: longhorn-vol-pv
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
    volumeHandle: test-volume
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: longhorn-vol-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
  volumeName: longhorn-vol-pv
  storageClassName: longhorn
---
apiVersion: v1
kind: Pod
metadata:
  name: volume-pv-test
  namespace: default
spec:
  restartPolicy: Always
  containers:
  - name: volume-pv-test
    image: nginx:stable-alpine
    imagePullPolicy: IfNotPresent
    livenessProbe:
      exec:
        command:
          - ls
          - /data/lost+found
      initialDelaySeconds: 5
      periodSeconds: 5
    volumeMounts:
    - name: vol
      mountPath: /data
    ports:
    - containerPort: 80
  volumes:
  - name: vol
    persistentVolumeClaim:
      claimName: longhorn-vol-pvc
```

### Dynamic PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: longhorn-volv-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: longhorn
  resources:
    requests:
      storage: 2Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: volume-test
  namespace: default
spec:
  restartPolicy: Always
  containers:
  - name: volume-test
    image: nginx:stable-alpine
    imagePullPolicy: IfNotPresent
    livenessProbe:
      exec:
        command:
          - ls
          - /data/lost+found
      initialDelaySeconds: 5
      periodSeconds: 5
    volumeMounts:
    - name: volv
      mountPath: /data
    ports:
    - containerPort: 80
  volumes:
  - name: volv
    persistentVolumeClaim:
      claimName: longhorn-volv-pvc
```

## Exposing the dashboard with Traefik

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: longhorn-urbaman
  namespace: longhorn-system
spec:
  # Certificate will be valid for these domain names
  dnsNames:
  - longhorn.urbaman.it
  # Reference our issuer
  # As it's a ClusterIssuer, it can be in a different namespace
  issuerRef:
    kind: ClusterIssuer
    name: cert-manager-acme-issuer
  # Secret that will be created with our certificate and private keys
  secretName: longhorn-urbaman
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: longhorn-dashboard-basic-auth
  namespace: longhorn-system
spec:
  basicAuth:
    secret: longhorn-dashboard-basic-auth
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: longhorn-dashboard-https-redirect
  namespace: longhorn-system
spec:
  redirectScheme:
    scheme: https
    permanent: true
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: longhorn-dashboard-security
  namespace: longhorn-system
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
  name: longhorn-dashboard-transport
  namespace: longhorn-system
spec:
  serverName: alertmanager
  insecureSkipVerify: true
---
apiVersion: traefik.containo.us/v1alpha1
kind: TLSOption
metadata:
  name: longhorn-dashboard-tlsoptions
  namespace: longhorn-system
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
  name: longhorn-dashboard-basic-auth
  namespace: longhorn-system
data:
  users: |
    YWRtaW46JGFwcjEkVmhXclBDUTIkZkFwRWVBMkhzODFMVkhEa3p1bmNoMQoK
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: longhorn-dashboard-websecure
  namespace: longhorn-system
spec:
  entryPoints:
    - websecure
  routes:
    - kind: Rule
      match: Host(`longhorn.urbaman.it`)
      services:
      - name: longhorn-frontend
        port: 80
        serversTransport: longhorn-dashboard-transport
      middlewares:
        - name: longhorn-dashboard-basic-auth
        - name: longhorn-dashboard-security
  tls:
    secretName: longhorn-urbaman
    options:
      name: longhorn-dashboard-tlsoptions
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: longhorn-dashboard-web
  namespace: longhorn-system
spec:
  entryPoints:
    - web
  routes:
    - kind: Rule
      match: Host(`longhorn.urbaman.it`)
      services:
      - name: longhorn-frontend
        port: 80
      middlewares:
        - name: longhorn-dashboard-https-redirect
```
