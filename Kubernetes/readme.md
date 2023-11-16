# Kubernetes Cluser

## Cluster infrastructure

Here is the Cluster infrastructure we will deploy

- 3x controllers: k8cp1, k8cp2, k8cp3
- 3x workers: k8w1, k8w2, k8w3
- 3x etcd cluster: etcd1, etcd2, etcd3
- 3x haproxy+keepalive: haproxy1, haproxy2, haproxy3
- 3x glusterfs cluster: nfs1, nfs22, nfs3

| Hostname               | CPU | RAM  | Disk                     | System             | Role                              | IP         |
| ---------------------- | --- | ---- | ------------------------ | ------------------ | --------------------------------- | ---------- |
| nfs1.domain.com, gluster1.domain.com | 8   | 16GB | 1x16GB (OS), 2x5TB (HDD - volume1), 2x2TB  (SDD - volume2) | Ubuntu 22.04       | GlusterFS node 1                  | 10.0.50.21, 10.0.70.21 (gluster storage) |
| nfs2.domain.com, gluster2.domain.com | 8   | 16GB | 1x16GB (OS), 2x5TB (HDD - volume1), 2x2TB  (SDD - volume2) | Ubuntu 22.04       | GlusterFS node 2                  | 10.0.50.22, 10.0.70.22 (gluster storage) |
| nfs3.domain.com, gluster3.domain.com | 8   | 16GB | 1x16GB (OS), 2x5TB (HDD - volume1), 2x2TB  (SDD - volume2) | Ubuntu 22.04       | GlusterFS node 3                  | 10.0.50.23, 10.0.70.23 (gluster storage) |
| k8cp1.domain.com       | 8   | 32GB | 32BG (OS), 2x200GB (longhorn)           | Ubuntu 22.04       | Kubernetes control manager node 1 | 10.0.50.51, 10.0.90.51 (longhorn storage) |
| k8cp2.domain.com       | 8   | 32GB | 32BG (OS), 2x200GB (longhorn)           | Ubuntu 22.04       | Kubernetes control manager node 2 | 10.0.50.52, 10.0.90.52 (longhorn storage) |
| k8cp3.domain.com       | 8   | 32GB | 32BG (OS), 2x200GB (longhorn)           | Ubuntu 22.04       | Kubernetes control manager node 3 | 10.0.50.53, 10.0.90.53 (longhorn storage) |
| k8w1.domain.com        | 8   | 32GB | 32BG (OS), 2x200GB (longhorn)           | Ubuntu 22.04       | Kubernetes worker node 1          | 10.0.50.54, 10.0.90.54 (longhorn storage) |
| k8w2.domain.com        | 8   | 32GB | 32BG (OS), 2x200GB (longhorn)           | Ubuntu 22.04       | Kubernetes worker node 2          | 10.0.50.55, 10.0.90.55 (longhorn storage) |
| k8w3.domain.com        | 8   | 32GB | 32BG (OS), 2x200GB (longhorn)           | Ubuntu 22.04       | Kubernetes worker node 3          | 10.0.50.56, 10.0.90.56 (longhorn storage) |
| ectd1.domain.com       | 4   | 8GB  | 12GB                     | Ubuntu 22.04       | Etcd cluster node 1               | 10.0.50.41 |
| etcd2.domain.com       | 4   | 8GB  | 12GB                     | Ubuntu 22.04       | Etcd cluster node 2               | 10.0.50.42 |
| etcd3.domain.com       | 4   | 8GB  | 12GB                     | Ubuntu 22.04       | Etcd cluster node 3               | 10.0.50.43 |
| haproxy1.domain.com    | 2   | 2GB  | 12GB                     | Ubuntu 22.04       | Keepalive Master/Haproxy node 1   | 10.0.50.61 |
| haproxy2.domain.com    | 2   | 2GB  | 12GB                     | Ubuntu 22.04       | Keepalive Backup/Haproxy node 2   | 10.0.50.62 |
| haproxy3.domain.com    | 2   | 2GB  | 12GB                     | Ubuntu 22.04       | Keepalive Backup/Haproxy node 3   | 10.0.50.63 |
| k8cp.domain.com, nfs.domain.com | N/A | N/A  | N/A                      | N/A                | Keepalive VIP IP                  | 10.0.50.64 |

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
5. [Second network with Multus and Whereabouts](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Multus)
6. [Storage with longhorn](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Storage/Longhorn)
7. [Storage with NFS](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Storage/NFS)
8. [Cloudflare Operator (DynDNS, DNS - deprecated)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cloudflare-Operator)
9. [Monitoring with Prometheus-stack (Prometheus, Grafana)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack)
10. [Kured for automatic reboots](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Kured)
11. [Node feature discovery - not needed with Nvidia GPU operator](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Node-Feature-Discovery)
12. [Nvidia GPU operator](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Nvidia-GPU)
13. [Reloader (?)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Reloader)
14. [k8TZ](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/k8tz)
15. [Cert Manager for SSL certs](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cert-manager)
16. [Traefik for ingress and proxy](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Traefik)
17. [Crowdsec security WAF for Traefik](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Crowdsec)
18. [Kubernetes Dashboard](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Dashboard)
19. [Portainer managing dashboard](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Portainer)
20. [Datree secure the cluster and avoid misconfigurations - deprecated](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Datree)
21. [Groundcover for observability](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Groundcover)
22. [Kubescape for observability and security scan](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Kubescape)
23. [Uptime Kuma for service uptime monitoring](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Uptimekuma)
24. [Teleport for external access](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Teleport)
25. [Postgresql replica cluster with Pgadmin](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Postgresql)
26. [Mysql replica cluster with Proxysql and Phpmyadmin](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Mysql)
27. [Mariadb replica cluster with Proxysql and Phpmyadmin](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Mariadb)
28. [Redis replica cluster](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Redis)
29. [Dbgate DB web client](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Dbgate)
30. [Cloudbeaver DB web client](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Cloudbeaver)
31. [Monitoring Calico](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Calico)
32. [Monitoring Longhorn](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Storage/Longhorn)
33. [Monitoring External Etcd](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/ExternalEtcd)
34. [Monitoring Cert Manager](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Cert-manager)
35. [Monitoring Kured](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Kured)
36. [Monitoring Proxmox](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Proxmox-Monitoring)
37. [Monitoring Proxmox Backup Server](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Proxmox-Backup-Monitoring)
38. [Monitoring Haproxy](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Haproxy-Monitoring)
39. [Monitoring snmp (iDRAC, switch)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Prometheus-snmp)
40. [Monitoring Fritzbox](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Fritzbox-exporter)
41. [Monitoring Glusterfs](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Storage/Glusterfs)
42. [Monitoring Ceph](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Storage/Ceph)
43. [Monitoring NFS](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Storage/NFS-server)
44. [Monitoring UptimeKuma](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Uptime-kuma)
45. [Monitoring kubescape](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Kubescape)
46. [Monitoring Postgresql](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Database/Postgresql)
47. [Monitoring Mysql](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Database/Mysql)
48. [Monitoring Redis](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Database/Redis)
49. [Monitoring Crowdsec](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Crowdsec)

### Deploying the services

1. [Heimdall homelab dashboard](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Heimdall-dashboard)
2. [Homer homelab dashboard](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Homer)
3. [it-tools](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/It-tools)
4. [Gethomepage](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Gethomepage)
5. [Drawio](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Drawio)
6. [Actual Budget for financial budgeting](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/ActualBudget)
7. [Stirling PDF for PDF Tools](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Stirling-PDF)
8. [FireflyIII](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/FireflyIII)
9. [Nextcloud](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Nextcloud)
10. Calibre+Calibre-Web
