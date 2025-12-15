# Microceph

## Prerequisites

At least three nodes, with at least two dedicated drives

## Cluster bootstrap

On all nodes:

```bash
sudo snap install microceph
sudo snap refresh --hold microceph
```

On first node:

```bash
sudo microceph cluster bootstrap --public-network  10.0.40.0/24 --cluster-network 10.0.30.0/24
sudo microceph cluster add <node 2>
sudo microceph cluster add <node 3>
```

On all other nodes:

```bash
sudo microceph cluster join <join token>
```

## Add OSDs

On all nodes, add the dedicated drives to the cluster, and set the replica count to 2 (being an homelab)

```bash
sudo microceph disk add <path to disk> --wipe
sudo microceph.ceph config set global osd_pool_default_size 2
```

## Create an RBD pool for images (k8s volumes)

```bash
sudo ceph osd pool create block_pool
sudo ceph osd lspools
sudo rbd pool init block_pool
```

## Dashboard enablement

```bash
microceph.ceph config set mgr mgr/dashboard/ssl false
sudo microceph.ceph mgr module enable dashboard
echo -n <password> | sudo tee /var/snap/microceph/current/conf/password.txt > /dev/null
sudo microceph.ceph dashboard ac-user-create -i /var/snap/microceph/current/conf/password.txt admin administrator
rm /var/snap/microceph/current/conf/password.txt
```

Go to any mgr ip, port 8080 in http only

## Eventually deploy a proxy for SSL

Example through traefik in kubernetes using file provider for loadbalancing and healthcheck between ceph nodes (the feature does not work in kubernetes ingress or ingressroute crd)

1. Enable file provider in the traefik helm chart
2. Enable the persistence pvc mounted in the default /data directory
3. Enable certresolver (see treafik deployment for clouflare) with /data as the directory storage for ssl certs
4. After the deployment, edit the traefik deployment and force the file provider directory in /data/rules
5. If you use NFS as storageclass, give the right permissions on the pvc created
6. Create the ceph-dashboard-router.yaml file in the /rules directory in the pvc
7. Deploy the ig-ceph-dashboard.yaml (if you want to keep middlewares and tlsoptions on kubernetescrd instead of file)
8. Rollout restart the traefik deployment, and/or delete the traefik pod if the deployment has already been restarted after the customization

## Enabling prometheus

ceph mgr module enable prometheus

You'll find the metrics on http://<IP>:9283/metrics, you can setup the scraping and monitoring in kubernetes (rancher or kube-prometheus-stack) following the ceph monitoring guide
