# Kubernetes Cluser

## Cluster infrastructure

Here is the Cluster infrastructure we will deploy

- 3x controllers: k8cp1, k8cp2, k8cp3
- 3x workers: k8w1, k8w2, k8w3
- 3x etcd cluster: etcd1, etce2, etcd3
- 3x haproxy+keepalive: haproxy1, haproxy2, haproxy3
- 3x glusterfs cluster: truenas1, truenas2, truenas3
- 1x glusterfs clusater manager: truecommand

| Hostname               | CPU | RAM  | Disk                     | System             | Role                              | IP         |
| ---------------------- | --- | ---- | ------------------------ | ------------------ | --------------------------------- | ---------- |
| truenas1.urbaman.it    | 4   | 16GB | 2x32GB (OS), 2x5TB (GFS) | TrueNAS SCALE      | GlusterFS node 1                  | 10.0.50.21 |
| truenas2.urbaman.it    | 4   | 16GB | 2x32GB (OS), 2x5TB (GFS) | TrueNAS SCALE      | GlusterFS node 2                  | 10.0.50.22 |
| truenas3.urbaman.it    | 4   | 16GB | 2x32GB (OS), 2x5TB (GFS) | TrueNAS SCALE      | GlusterFS node 3                  | 10.0.50.23 |
| truecommand.urbaman.it | 2   | 4GB  | 32GB                     | Debian/Truecommand | GlusterFS cluster manager         | 10.0.50.24 |
| k8cp1.urbaman.it       | 8   | 32GB | 16BG                     | Ubuntu 22.04       | Kubernetes control manager node 1 | 10.0.50.31 |
| k8cp2.urbaman.it       | 8   | 32GB | 16BG                     | Ubuntu 22.04       | Kubernetes control manager node 2 | 10.0.50.32 |
| k8cp3.urbaman.it       | 8   | 32GB | 16BG                     | Ubuntu 22.04       | Kubernetes control manager node 3 | 10.0.50.33 |
| k8w1.urbaman.it        | 8   | 32GB | 16BG                     | Ubuntu 22.04       | Kubernetes worker node 1          | 10.0.50.34 |
| k8w2.urbaman.it        | 8   | 32GB | 16BG                     | Ubuntu 22.04       | Kubernetes worker node 2          | 10.0.50.35 |
| k8w3.urbaman.it        | 8   | 32GB | 16BG                     | Ubuntu 22.04       | Kubernetes worker node 3          | 10.0.50.36 |
| ectd1.urbaman.it       | 2   | 4GB  | 16GB                     | Ubuntu 22.04       | Etcd cluster node 1               | 10.0.50.41 |
| etcd2.urbaman.it       | 2   | 4GB  | 16GB                     | Ubuntu 22.04       | Etcd cluster node 2               | 10.0.50.42 |
| etcd3.urbaman.it       | 2   | 4GB  | 16GB                     | Ubuntu 22.04       | Etcd cluster node 3               | 10.0.50.43 |
| haproxy1.urbaman.it    | 2   | 4GB  | 16GB                     | Ubuntu 22.04       | Keepalive Master/Haproxy node 1   | 10.0.50.61 |
| haproxy2.urbaman.it    | 2   | 4GB  | 16GB                     | Ubuntu 22.04       | Keepalive Backup/Haproxy node 2   | 10.0.50.62 |
| haproxy3.urbaman.it    | 2   | 4GB  | 16GB                     | Ubuntu 22.04       | Keepalive Backup/Haproxy node 3   | 10.0.50.63 |
| k8cp.urbaman.it        | N/A | N/A  | N/A                      | N/A                | Keepalive VIP IP                  | 10.0.50.64 |

## Cluster deployment

Here are the detailed steps to deploy our homelab cluster

- [Preparing the machines](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/01-Prepare-Machines)
