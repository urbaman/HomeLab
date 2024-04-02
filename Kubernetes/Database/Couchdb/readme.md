# Installation

Add the repo and install

```bash
helm repo add couchdb https://apache.github.io/couchdb-helm
helm repo update
helm upgrade -i couchdb couchdb/couchdb --set persistentVolume.enabled=true --set persistentVolume.storageClass=rook-ceph-nvme2tb --set couchdbConfig.chttpd.require_valid_user=true --set prometheusPort.enabled=true --set couchdbConfig.couchdb.uuid=90486a5d-b089-4356-8c1a-4f99fe63cb13 --set labels.app.kubernetes.io/name=couchdb --namespace couchdb --create-namespace
```

Get the password for the admin user

```bash
kubectl get secret -n couchdb couchdb-couchdb -o go-template='{{ .data.adminPassword }}' | base64 -d
```

Connect to the Fauxton GUI to check the cluster and begin using it: [http://couchdb-couchdb.couchdb.svc.cluster.local:5984/_utils/#/](http://couchdb-couchdb.couchdb.svc.cluster.local:5984/_utils/#/)

Add Traefik:

```bash
kubectl apply -n cluchdb-traefik.yaml
```

Now you can connect to couchdb.domain.com to contact the cluster
