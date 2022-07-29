# Portainer

To deploy Portainer, get the yaml manifest:

wget <https://raw.githubusercontent.com/portainer/k8s/master/deploy/manifests/portainer/portainer.yaml>

Add a persistent volume and a persistent volume claim to the manifest just after the service account (needs to be after the namespace creation, before the actual deploy creation), and reference the claim in the volume definition:

```bash
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

```bash
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
