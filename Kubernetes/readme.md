# Kubernetes Cluser

## Cluster infrastructure

Here is the Cluster infrastructure we will deploy on Proxmox

- 3x controllers: k8cp1, k8cp2, k8cp3
- 9x workers: k8w1, k8w2, k8w3, k8w4, k8w5, k8w6, k8w7, k8w8, k8w9
- 3x etcd cluster: etcd1, etcd2, etcd3
- 3x haproxy+keepalive: haproxy1, haproxy2, haproxy3

| Hostname               | CPU | RAM  | Disk                     | System             | Role                              | IP         |
| ---------------------- | --- | ---- | ------------------------ | ------------------ | --------------------------------- | ---------- |
| k8cp1.domain.com       | 4   | 16GB | 100GB           | Ubuntu 24.04       | Kubernetes control manager node 1 | 10.0.50.51 |
| k8cp2.domain.com       | 4   | 16GB | 100GB           | Ubuntu 24.04       | Kubernetes control manager node 2 | 10.0.50.52 |
| k8cp3.domain.com       | 4   | 16GB | 100GB           | Ubuntu 24.04       | Kubernetes control manager node 3 | 10.0.50.53 |
| k8w1.domain.com        | 8   | 32GB | 100GB           | Ubuntu 24.04       | Kubernetes worker node 1          | 10.0.50.81 |
| k8w2.domain.com        | 8   | 32GB | 100GB           | Ubuntu 24.04       | Kubernetes worker node 2          | 10.0.50.82 |
| k8w3.domain.com        | 8   | 32GB | 100GB           | Ubuntu 24.04       | Kubernetes worker node 3          | 10.0.50.83 |
| k8w1.domain.com        | 8   | 32GB | 100GB           | Ubuntu 24.04       | Kubernetes worker node 4          | 10.0.50.84 |
| k8w5.domain.com        | 8   | 32GB | 100GB           | Ubuntu 24.04       | Kubernetes worker node 5          | 10.0.50.85 |
| k8w6.domain.com        | 8   | 32GB | 100GB           | Ubuntu 24.04       | Kubernetes worker node 6          | 10.0.50.86 |
| k8w7.domain.com        | 8   | 32GB | 100GB           | Ubuntu 24.04       | Kubernetes worker node 7          | 10.0.50.87 |
| k8w8.domain.com        | 8   | 32GB | 100GB           | Ubuntu 24.04       | Kubernetes worker node 8          | 10.0.50.88 |
| k8w9.domain.com        | 8   | 32GB | 100GB           | Ubuntu 24.04       | Kubernetes worker node 9          | 10.0.50.89 |
| etcd1.domain.com       | 4   | 8GB  | 12GB                     | Ubuntu 24.04       | Etcd cluster node 1               | 10.0.50.41 |
| etcd2.domain.com       | 4   | 8GB  | 12GB                     | Ubuntu 24.04       | Etcd cluster node 2               | 10.0.50.42 |
| etcd3.domain.com       | 4   | 8GB  | 12GB                     | Ubuntu 24.04       | Etcd cluster node 3               | 10.0.50.43 |
| haproxy1.domain.com    | 2   | 2GB  | 12GB                     | Ubuntu 24.04       | Keepalive Master/Haproxy node 1   | 10.0.50.61 |
| haproxy2.domain.com    | 2   | 2GB  | 12GB                     | Ubuntu 24.04       | Keepalive Backup/Haproxy node 2   | 10.0.50.62 |
| haproxy3.domain.com    | 2   | 2GB  | 12GB                     | Ubuntu 24.04       | Keepalive Backup/Haproxy node 3   | 10.0.50.63 |
| k8cp.domain.com, nfs.domain.com | N/A | N/A  | N/A                      | N/A                | Keepalive VIP IP                  | 10.0.50.64 |

## Cluster deployment

Here are the detailed steps to deploy our homelab cluster

### Creating the Cluster

1. [Preparing the machines](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/01-Prepare-Machines)
2. [External Etcd](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/02-External-Etcd)
3. [High Availability](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/03-High-Availability)
4. [Kubernetes](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/04-Kubernetes)

### Storage planning

1. [Storage planning](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Storage)

### Deploying the tools

#### Automation

1. [GitLab](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Autometion/Gitlab)
2. [ArgoCD](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Autometion/ArgoCD)
3. [Hashcorp Vault](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Autometion/HashicorpVault)
4. [External Secrets](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Autometion/ExternalSecrets)

#### Coding

1. [VScode (code-server)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Coding/CodeServer)

#### Backup

1. [Velero](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/K8sBackup/Velero)

#### The other tools

1. [Helm](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Helm)
2. [Metrics Server](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Metrics-Server)
3. [Cert Manager for SSL certs](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cert-manager)
4. [Storage with Glusterfs](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Storage/Glusterfs)
5. [Storage with Rook (backed by Proxmox Ceph)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Storage/Rook)
6. [Second network with Multus and Whereabouts](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Multus)
7. [Storage with longhorn](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Storage/Longhorn)
8. [Storage with NFS](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Storage/NFS)
9. [Cloudflare Operator (DynDNS, DNS - deprecated)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cloudflare-Operator)
10. [Monitoring with Prometheus-stack (Prometheus, Grafana)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack)
11. [Metallb LoadBalancer](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Metallb)
12. [Kured for automatic reboots](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Kured)
13. [k8TZ](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/k8tz)
14. [Node feature discovery - not needed with Nvidia GPU operator](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Node-Feature-Discovery)
15. [Nvidia GPU operator](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Nvidia-GPU)
16. [Intel GPU Operator](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Intel-GPU)
17. [ClamAV antivirus server](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/ClamAV)
18. [Reloader (?)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Reloader)
19. [Traefik for ingress and proxy](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Traefik)
20. [Traefik for External Service: Shared Hosting](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Hosting)
21. [Crowdsec security WAF for Traefik](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Crowdsec)
22. [Kubernetes Dashboard](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Dashboard)
23. [Portainer managing dashboard](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Portainer)
24. [Datree secure the cluster and avoid misconfigurations - deprecated](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Datree)
25. [Kubetail - k8s log viewer](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Kubetail)
26. [Groundcover for observability](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Groundcover)
27. [Kubescape for observability and security scan](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Kubescape)
28. [Uptime Kuma for service uptime monitoring](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Uptimekuma)
29. [Teleport for external access](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Teleport)
30. [Postgresql replica cluster with Pgadmin](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Postgresql)
31. [Mysql replica cluster with Proxysql and Phpmyadmin](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Mysql)
32. [Mariadb replica cluster with Proxysql and Phpmyadmin](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Mariadb)
33. [Redis replica cluster](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Redis)
34. [Dbgate DB web client](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Dbgate)
35. [Cloudbeaver DB web client](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Cloudbeaver)
36. [Memcached](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Memcached)
37. [MinIO for S3 object Storage](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Storage/MinIO)
38. [Monitoring Calico](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Calico)
39. [Monitoring Longhorn](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Storage/Longhorn)
40. [Monitoring External Etcd](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/ExternalEtcd)
41. [Monitoring Cert Manager](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Cert-manager)
42. [Monitoring Kured](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Kured)
43. [Monitoring Proxmox](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Proxmox-Monitoring)
44. [Monitoring Proxmox Backup Server](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Proxmox-Backup-Monitoring)
45. [Monitoring Haproxy](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Haproxy-Monitoring)
46. [Monitoring snmp (iDRAC, switch)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Prometheus-snmp)
47. [Monitoring Fritzbox](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Fritzbox-exporter)
48. [Monitoring Glusterfs](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Storage/Glusterfs)
49. [Monitoring Ceph](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Storage/Ceph)
50. [Monitoring NFS](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Storage/NFS-server)
51. [Monitoring UptimeKuma](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Uptime-kuma)
52. [Monitoring kubescape](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Kubescape)
53. [Monitoring Postgresql](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Database/Postgresql)
54. [Monitoring Mysql](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Database/Mysql)
55. [Monitoring Redis](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Database/Redis)
56. [Monitoring Memcached](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Database/Memcached)
57. [Monitoring Crowdsec](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Crowdsec)

### Deploying the services

1. [Heimdall homelab dashboard](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Heimdall-dashboard)
2. [Homer homelab dashboard](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Homer)
3. [it-tools](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/It-tools)
4. [Homepage](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Homepage)
5. [Netbox](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Netbox)
6. [Drawio](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Drawio)
7. [Actual Budget for financial budgeting](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/ActualBudget)
8. [Stirling PDF for PDF Tools](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Stirling-PDF)
9. [FireflyIII](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/FireflyIII)
10. [Collabora online](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Collabora)
11. [Nextcloud](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Nextcloud)
12. [Media Server: Transmission, Sonarr, Radarr, Readarr, Lidarr, Bazarr, Unpackerr, Prowlarr, Plex, Overseerr](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Media-Server)
13. [Calibre server and Calibre web](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Media-Server)
14. [BOINC shared computing](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Computing/Boinc)
