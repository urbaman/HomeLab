# Kubernetes: more tools

## Metallb

### Preparation

If you’re using kube-proxy in IPVS mode, since Kubernetes v1.14.2 you have to enable strict ARP mode.

Note, you don’t need this if you’re using kube-router as service-proxy because it is enabling strict ARP by default.

You can achieve this by editing kube-proxy config in current cluster:

```bash
kubectl edit configmap -n kube-system kube-proxy
```

and set:

```bash
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
ipvs:
  strictARP: true
```

You can also add this configuration snippet to your kubeadm-config, just append it with --- after the main configuration.

If you are trying to automate this change, these shell snippets may help you:

```bash
# see what changes would be made, returns nonzero returncode if different
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl diff -f - -n kube-system

# actually apply the changes, returns nonzero returncode on errors only
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl apply -f - -n kube-system
```

### Installation By Manifest

To install MetalLB, apply the manifest:

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/metallb.yaml
```

## Storage (GlusterFS)

### glusterfs-client

Fist you need to install the glusterfs-client package on your nodes. The client is used by the kubernetes scheduler to create the gluster volumes.

```bash
$ sudo apt install gluster-client
```

### Discovering GlusterFS in Kubernetes:

GlusterFS cluster should be discovered in the Kubernetes cluster. To do that, you need to add an Endpoints object that points to the servers of the GlusterFS cluster.

```bash
apiVersion: v1
kind: Endpoints
metadata:
  name: glusterfs-cluster
  labels:
    storage.k8s.io/name: glusterfs
    storage.k8s.io/part-of: kubernetes-complete-reference
    storage.k8s.io/created-by: ssbostan
subsets:
  - addresses:
      - ip: 10.0.50.21
        hostname: truenas1.urbaman.it
      - ip: 10.0.50.22
        hostname: truenas2.urbaman.it
      - ip: 10.0.50.23
        hostname: truenas3.urbaman.it
    ports:
      - port: 1
```

To persist the Gluster endpoints, you also need to create a relative service.

```bash
apiVersion: v1
kind: Service
metadata:
  name: glusterfs-cluster 
spec:
  ports:
  - port: 1
```

### Using GlusterFS in Kubernetes:

Create the desired path in the glusterfs volume (HDD5T/path/... in my situation)

##### Method 1 — Connecting to GlusterFS directly with Pod manifest:

To connect to the GlusterFS volume directly with Pod manifest, use the GlusterfsVolumeSource in the Pod. Here is an example (create the path in the glusterfs volume - HDD5T in my situation):

```bash
apiVersion: v1
kind: Pod
metadata:
  name: test
  labels:
    app.kubernetes.io/name: alpine
    app.kubernetes.io/part-of: kubernetes-complete-reference
    app.kubernetes.io/created-by: ssbostan
spec:
  containers:
    - name: alpine
      image: alpine:latest
      command:
        - touch
        - /data/test
      volumeMounts:
        - name: glusterfs-volume
          mountPath: /data
  volumes:
    - name: glusterfs-volume
      glusterfs:
        endpoints: glusterfs-cluster
        path: HDD5T/path/...
        readOnly: no
```

##### Method 2 — Connecting using the PersistentVolume resource:

To create the PersistentVolume object for the GlusterFS volume, use the following manifest. The storage size does not take any effect. Be sure to name the claim you'll bind it to.

```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: glusterfs-path
  namespace: project-namespace
spec:
  accessModes:
    - ReadWriteOnce
    - ReadOnlyMany
    - ReadWriteMany
  capacity:
    storage: 10Gi
  storageClassName: ""
  persistentVolumeReclaimPolicy: Recycle
  volumeMode: Filesystem
  glusterfs:
    endpoints: glusterfs-cluster
    path: HDD5T/path/...
    readOnly: no
  claimRef:
    name: glusterfs-path-pvc
    namespace: project-namespace
```

The create the relative PersistentVolumeClaim (be sure to name the PersistentVolume to Claim:

```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: glusterfs-path-pvc
    namespace: project-namespace
spec:
    storageClassName: ""
    volumeName: glusterfs-path
```

And then, use the claim as a volume in the pods:

```bash
apiVersion: v1
kind: Pod
metadata:
    name: mypod
    namespace: project-namespace
spec:
    containers:
      - name: myfrontend
        image: nginx
        volumeMounts:
        - mountPath: "/var/www/html"
          name: mypd
    volumes:
      - name: mypd
        persistentVolumeClaim:
          claimName: glusterfs-path-pvc
```

## Helm

### From Apt (Debian/Ubuntu)

Members of the Helm community have contributed a Helm package for Apt. This package is generally up to date.

```bash
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

### From Snap

The Snapcrafters community maintains the Snap version of the Helm package:

```bash
sudo snap install helm --classic
```

## Cloudflare DynDNS

https://docs.cf.containeroo.ch/

## Monitoring

https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack

## Traefik
https://www.padok.fr/en/blog/traefik-kubernetes-certmanager

https://www.howtogeek.com/devops/how-to-install-kubernetes-cert-manager-and-configure-lets-encrypt/

https://www.youtube.com/watch?v=dEAtD9PVr_Q

https://dev.to/bgalvao/traefik-lets-encrypt-cloudflare-36fj
