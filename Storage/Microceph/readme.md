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

On all nodes, add the dedicated drives to the cluster

```bash
sudo microceph disk add <path to disk> --wipe
```

## Dashboard enablement

```bash
microceph.ceph config set mgr mgr/dashboard/ssl false
sudo microceph.ceph mgr module enable dashboard
echo -n <password> | sudo tee /var/snap/microceph/current/conf/password.txt > /dev/null
sudo microceph.ceph dashboard ac-user-create -i /var/snap/microceph/current/conf/password.txt admin administrator
rm /var/snap/microceph/current/conf/password.txt
```

Eventually deploy a proxy (traefik ingressroute example provided ig-ceph-dashboard.yaml for K8s or microk8s)
