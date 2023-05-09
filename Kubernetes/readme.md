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
| nfs1.urbaman.it        | 4   | 16GB | 1x16GB (OS), 2x5TB (HDD - volume1), 2x2TB  (SDD - volume2) | Ubuntu 22.04       | GlusterFS node 1                  | 10.0.50.21 |
| nfs.urbaman.it         | 4   | 16GB | 1x16GB (OS), 2x5TB (HDD - volume1), 2x2TB  (SDD - volume2) | Ubuntu 22.04       | GlusterFS node 2                  | 10.0.50.22 |
| nfs.urbaman.it         | 4   | 16GB | 1x16GB (OS), 2x5TB (HDD - volume1), 2x2TB  (SDD - volume2) | Ubuntu 22.04       | GlusterFS node 3                  | 10.0.50.23 |
| k8cp1.urbaman.it       | 8   | 32GB | 16BG (OS), 2x32GB (longhorn)           | Ubuntu 22.04       | Kubernetes control manager node 1 | 10.0.50.51 |
| k8cp2.urbaman.it       | 8   | 32GB | 16BG (OS), 2x32GB (longhorn)           | Ubuntu 22.04       | Kubernetes control manager node 2 | 10.0.50.52 |
| k8cp3.urbaman.it       | 8   | 32GB | 16BG (OS), 2x32GB (longhorn)           | Ubuntu 22.04       | Kubernetes control manager node 3 | 10.0.50.53 |
| k8w1.urbaman.it        | 8   | 32GB | 16BG (OS), 2x32GB (longhorn)           | Ubuntu 22.04       | Kubernetes worker node 1          | 10.0.50.54 |
| k8w2.urbaman.it        | 8   | 32GB | 16BG (OS), 2x32GB (longhorn)           | Ubuntu 22.04       | Kubernetes worker node 2          | 10.0.50.55 |
| k8w3.urbaman.it        | 8   | 32GB | 16BG (OS), 2x32GB (longhorn)           | Ubuntu 22.04       | Kubernetes worker node 3          | 10.0.50.56 |
| ectd1.urbaman.it       | 2   | 4GB  | 16GB                     | Ubuntu 22.04       | Etcd cluster node 1               | 10.0.50.41 |
| etcd2.urbaman.it       | 2   | 4GB  | 16GB                     | Ubuntu 22.04       | Etcd cluster node 2               | 10.0.50.42 |
| etcd3.urbaman.it       | 2   | 4GB  | 16GB                     | Ubuntu 22.04       | Etcd cluster node 3               | 10.0.50.43 |
| haproxy1.urbaman.it    | 2   | 4GB  | 16GB                     | Ubuntu 22.04       | Keepalive Master/Haproxy node 1   | 10.0.50.61 |
| haproxy2.urbaman.it    | 2   | 4GB  | 16GB                     | Ubuntu 22.04       | Keepalive Backup/Haproxy node 2   | 10.0.50.62 |
| haproxy3.urbaman.it    | 2   | 4GB  | 16GB                     | Ubuntu 22.04       | Keepalive Backup/Haproxy node 3   | 10.0.50.63 |
| k8cp.urbaman.it        | N/A | N/A  | N/A                      | N/A                | Keepalive VIP IP                  | 10.0.50.64 |

## Cluster deployment

Here are the detailed steps to deploy our homelab cluster

### Creating the Cluster

1. [Preparing the machines](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/01-Prepare-Machines)
2. [External Etcd](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/02-External-Etcd)
3. [High Availability](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/03-High-Availability)
4. [Kubernetes](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/04-Kubernetes)

### Deploying the tools

1. [Helm](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Helm)
2. [Metallb LoadBalancer](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Metallb)
3. [Storage with Glusterfs](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Glusterfs)
4. [Cloudflare Operator (DynDNS, DNS)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cloudflare-Operator)
5. [Monitoring with Prometheus-stack (Prometheus, Grafana)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack)
6. [Monitoring external services (Proxmox)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Proxmox-Monitoring)
7. [Traefik for ingress and proxy](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Traefik)
8. [Kubernetes Dashboard](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Dashboard)
9. [Metrics Server](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Metrics-Server)
10. [Portainer managing dashboard](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Portainer)
11. [Kured for automatic reboots](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Kured)

### Deploying the services

1. [Nextcloud](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Nextcloud)
