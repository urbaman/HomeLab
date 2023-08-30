# Kubernetes Cluser

## Cluster infrastructure

Here is the Cluster infrastructure we will deploy

- 3x controllers: k8cp1, k8cp2, k8cp3
- 3x workers: k8w1, k8w2, k8w3
- 3x etcd cluster: etcd1, etce2, etcd3
- 3x haproxy+keepalive: haproxy1, haproxy2, haproxy3
- 3x glusterfs cluster: nfs1, nfs22, nfs3

| Hostname               | CPU | RAM  | Disk                     | System             | Role                              | IP         |
| ---------------------- | --- | ---- | ------------------------ | ------------------ | --------------------------------- | ---------- |
| nfs1.urbaman.it, gluster1.urbaman.it | 8   | 16GB | 1x16GB (OS), 2x5TB (HDD - volume1), 2x2TB  (SDD - volume2) | Ubuntu 22.04       | GlusterFS node 1                  | 10.0.50.21, 10.0.70.21 (gluster storage) |
| nfs2.urbaman.it, gluster2.urbaman.it | 8   | 16GB | 1x16GB (OS), 2x5TB (HDD - volume1), 2x2TB  (SDD - volume2) | Ubuntu 22.04       | GlusterFS node 2                  | 10.0.50.22, 10.0.70.22 (gluster storage) |
| nfs3.urbaman.it, gluster3.urbaman.it | 8   | 16GB | 1x16GB (OS), 2x5TB (HDD - volume1), 2x2TB  (SDD - volume2) | Ubuntu 22.04       | GlusterFS node 3                  | 10.0.50.23, 10.0.70.23 (gluster storage) |
| k8cp1.urbaman.it       | 8   | 32GB | 32BG (OS), 2x200GB (longhorn)           | Ubuntu 22.04       | Kubernetes control manager node 1 | 10.0.50.51, 10.0.90.51 (longhorn storage) |
| k8cp2.urbaman.it       | 8   | 32GB | 32BG (OS), 2x200GB (longhorn)           | Ubuntu 22.04       | Kubernetes control manager node 2 | 10.0.50.52, 10.0.90.52 (longhorn storage) |
| k8cp3.urbaman.it       | 8   | 32GB | 32BG (OS), 2x200GB (longhorn)           | Ubuntu 22.04       | Kubernetes control manager node 3 | 10.0.50.53, 10.0.90.53 (longhorn storage) |
| k8w1.urbaman.it        | 8   | 32GB | 32BG (OS), 2x200GB (longhorn)           | Ubuntu 22.04       | Kubernetes worker node 1          | 10.0.50.54, 10.0.90.54 (longhorn storage) |
| k8w2.urbaman.it        | 8   | 32GB | 32BG (OS), 2x200GB (longhorn)           | Ubuntu 22.04       | Kubernetes worker node 2          | 10.0.50.55, 10.0.90.55 (longhorn storage) |
| k8w3.urbaman.it        | 8   | 32GB | 32BG (OS), 2x200GB (longhorn)           | Ubuntu 22.04       | Kubernetes worker node 3          | 10.0.50.56, 10.0.90.56 (longhorn storage) |
| ectd1.urbaman.it       | 4   | 8GB  | 12GB                     | Ubuntu 22.04       | Etcd cluster node 1               | 10.0.50.41 |
| etcd2.urbaman.it       | 4   | 8GB  | 12GB                     | Ubuntu 22.04       | Etcd cluster node 2               | 10.0.50.42 |
| etcd3.urbaman.it       | 4   | 8GB  | 12GB                     | Ubuntu 22.04       | Etcd cluster node 3               | 10.0.50.43 |
| haproxy1.urbaman.it    | 2   | 2GB  | 12GB                     | Ubuntu 22.04       | Keepalive Master/Haproxy node 1   | 10.0.50.61 |
| haproxy2.urbaman.it    | 2   | 2GB  | 12GB                     | Ubuntu 22.04       | Keepalive Backup/Haproxy node 2   | 10.0.50.62 |
| haproxy3.urbaman.it    | 2   | 2GB  | 12GB                     | Ubuntu 22.04       | Keepalive Backup/Haproxy node 3   | 10.0.50.63 |
| k8cp.urbaman.it, nfs.urbaman.it | N/A | N/A  | N/A                      | N/A                | Keepalive VIP IP                  | 10.0.50.64 |

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
3. [Metrics Server](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Metrics-Server)
4. [Storage with Glusterfs](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Glusterfs)
5. [Storage with longhorn](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Longhorn)
6. [Cloudflare Operator (DynDNS, DNS - deprecated)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cloudflare-Operator)
7. [Monitoring with Prometheus-stack (Prometheus, Grafana)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack)
8. [Monitoring external services 1 (Proxmox)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Proxmox-Monitoring)
9. [Monitoring external services 2 (Proxmox Backup Server)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Proxmox-Backup-Monitoring)
10. [Monitoring external services 3 (Haproxy)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Haproxy-Monitoring)
11. [Cert Manager for SSL certs](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cert-manager)
12. [Traefik for ingress and proxy](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Traefik)
13. [Kubernetes Dashboard](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Dashboard)
14. [Metrics Server](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Metrics-Server)
15. [Portainer managing dashboard](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Portainer)
16. [Kured for automatic reboots](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Kured)
17. [Datree secure the cluster and avoid misconfigurations](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Datree)
18. [Uptime Kuma for service uptime monitoring](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Uptimekuma)
19. [Teleport for external access](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Teleport)
20. [Nvidia GPU plugin](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Nvidia-GPU)

### Deploying the services

1. [Heimdall homelab dashboard](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Heimdall-dashboard)
2. [Homer homelab dashboard](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Homer)
3. [Nextcloud](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Nextcloud)
