# Kubernetes Cluster

## Cluster infrastructure (Kubeadm)

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
| etcd1.domain.com[^1]   | 4   | 8GB  | 12GB            | Ubuntu 24.04       | Etcd cluster node 1               | 10.0.50.41 |
| etcd2.domain.com[^1]   | 4   | 8GB  | 12GB            | Ubuntu 24.04       | Etcd cluster node 2               | 10.0.50.42 |
| etcd3.domain.com[^1]   | 4   | 8GB  | 12GB            | Ubuntu 24.04       | Etcd cluster node 3               | 10.0.50.43 |
| haproxy1.domain.com[^1]| 2   | 2GB  | 12GB            | Ubuntu 24.04       | Keepalive Master/Haproxy node 1   | 10.0.50.61 |
| haproxy2.domain.com[^1]| 2   | 2GB  | 12GB            | Ubuntu 24.04       | Keepalive Backup/Haproxy node 2   | 10.0.50.62 |
| haproxy3.domain.com[^1]| 2   | 2GB  | 12GB            | Ubuntu 24.04       | Keepalive Backup/Haproxy node 3   | 10.0.50.63 |
| k8cp.domain.com        | N/A | N/A  | N/A             | N/A                | Keepalive VIP IP                  | 10.0.50.64 |

[^1]: Etcd and haproxy servers only needed without KubeVIP for High Availability.

## Cluster Infrastructure (Microk8s)

- 3x nodes: mk8s1, mk8s2, mk8s3

| Hostname               | CPU | RAM  | Disk                     | System             | Role                              | IP         |
| ---------------------- | --- | ---- | ------------------------ | ------------------ | --------------------------------- | ---------- |
| mk8s1.domain.com       | 4   | 32GB | 150GB                    | Ubuntu 24.04       | Microk8s node 1                   | 10.0.100.51|
| mk8s2.domain.com       | 4   | 32GB | 150GB                    | Ubuntu 24.04       | Microk8s node 2                   | 10.0.100.52|
| mk8s3.domain.com       | 4   | 32GB | 150GB                    | Ubuntu 24.04       | Microk8s node 3                   | 10.0.100.53|
| mk8s4.domain.com       | 4   | 32GB | 150GB                    | Ubuntu 24.04       | Microk8s node 4                   | 10.0.100.54|
| mk8s5.domain.com       | 4   | 32GB | 150GB                    | Ubuntu 24.04       | Microk8s node 5                   | 10.0.100.55|
| mk8s6.domain.com       | 4   | 32GB | 150GB                    | Ubuntu 24.04       | Microk8s node 6                   | 10.0.100.56|
| mk8s.domain.com        | N/A | N/A  | N/A                      | N/A                | KubeVIP VIP IP                    | 10.0.100.50|

## Cluster deployment

Here are the detailed steps to deploy our homelab cluster

### Creating the Cluster

1. [Preparing the machines](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/01-Prepare-Machines)
2. [External Etcd](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/02-External-Etcd)
3. [High Availability](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/03-High-Availability)
4. [Kubernetes](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/04-Kubernetes)

### Storage planning

1. [Storage planning](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Storage)

### Command line tips

1. [Kubectl tips](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Kubectl)

### Deploying the tools

#### Automation

1. [GitLab](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Autometion/Gitlab)
2. [ArgoCD](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Autometion/ArgoCD)
3. [Hashcorp Vault](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Autometion/HashicorpVault)
4. [External Secrets](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Autometion/ExternalSecrets)
5. [Semaphore UI - Ansible Gui](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Autometion/Semaphore)

#### Identity Provider

1. [Authentik](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Authentik)

#### Coding

1. [VScode (code-server)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Coding/CodeServer)

#### Backup

1. [Velero](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/K8sBackup/Velero)

#### The other tools

1. [Helm](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Helm)
2. [Kyverno Admission Controller](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Kyverno)
3. [Metrics Server](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Metrics-Server)
4. [Descheduler](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Descheduler)
5. [Cert Manager for SSL certs](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cert-manager)
6. [Storage with Glusterfs](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Storage/Glusterfs)
7. [Storage with Rook (backed by Proxmox Ceph)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Storage/Rook)
8. [Second network with Multus and Whereabouts](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Multus)
9. [Storage with longhorn](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Storage/Longhorn)
10. [Storage with NFS](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Storage/NFS)
11. [Cloudflare Operator (DynDNS, DNS - deprecated)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cloudflare-Operator)
12. [Metallb LoadBalancer](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Metallb)
13. [Kured for automatic reboots](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Kured)
14. [k8TZ](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/k8tz)
15. [Node feature discovery - not needed with Nvidia GPU operator](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Node-Feature-Discovery)
16. [Nvidia GPU operator](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Nvidia-GPU)
17. [Intel GPU Operator](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Intel-GPU)
18. [ClamAV antivirus server](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/ClamAV)
19. [Reloader (?)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Reloader)
20. [Traefik for ingress and proxy](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Traefik)
21. [Traefik for External Service: Shared Hosting](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Hosting)
22. [Reflector](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Reflector)
23. [Crowdsec security WAF for Traefik](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Crowdsec)
24. [Kubernetes Dashboard](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Dashboard)
25. [Portainer managing dashboard](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Portainer)
26. [Datree secure the cluster and avoid misconfigurations - deprecated](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Datree)
27. [Kubetail - k8s log viewer](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Kubetail)
28. [Groundcover for observability](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Groundcover)
29. [Kubescape for observability and security scan](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Kubescape)
30. [Wazuh Security Scanner](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Wazuh)
31. [Uptime Kuma for service uptime monitoring](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Uptimekuma)
32. [Teleport for external access](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Teleport)
33. [Postgresql with Pgadmin](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Postgresql)
34. [Postgresql replica cluster with Pgadmin](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Postgresql-ha)
35. [Mysql standalone or cluster with Proxysql and Phpmyadmin](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Mysql)
36. [Mariadb standalone or replica cluster with Proxysql and Phpmyadmin](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Mariadb)
37. [Redis standalone or replica cluster](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Redis)
38. [Mongodb standalone or replica cluster](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Mongodb)
39. [Dbgate DB web client](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Dbgate)
40. [Cloudbeaver DB web client](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Cloudbeaver)
41. [Memcached](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Memcached)
42. [MinIO for S3 object Storage](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Storage/MinIO)
43. [Monitoring Calico](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Calico)
44. [Monitoring Longhorn](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Storage/Longhorn)
45. [Monitoring External Etcd](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/ExternalEtcd)
46. [Monitoring Cert Manager](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Cert-manager)
47. [Monitoring Kured](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Kured)
48. [Monitoring Proxmox](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Proxmox-Monitoring)
49. [Monitoring Proxmox Backup Server](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Proxmox-Backup-Monitoring)
50. [Monitoring Haproxy](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Haproxy-Monitoring)
51. [Monitoring snmp (iDRAC, switch)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Prometheus-snmp)
52. [Monitoring Fritzbox](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Fritzbox-exporter)
53. [Monitoring Glusterfs](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Storage/Glusterfs)
54. [Monitoring Ceph](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Storage/Ceph)
55. [Monitoring NFS](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Storage/NFS-server)
56. [Monitoring UptimeKuma](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Uptime-kuma)
57. [Monitoring kubescape](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Kubescape)
58. [Monitoring Postgresql](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Database/Postgresql)
59. [Monitoring Mysql](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Database/Mysql)
60. [Monitoring Redis](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Database/Redis)
61. [Monitoring Mongodb](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Database/Mongodb)
62. [Monitoring Memcached](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Database/Memcached)
63. [Monitoring Crowdsec](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Crowdsec)

### Deploying the services

1. [Heimdall homelab dashboard](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Heimdall-dashboard)
2. [Homer homelab dashboard](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Homer)
3. [Roundcube](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Roundcube)
4. [it-tools](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/It-tools)
5. [Cyberchef](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cyberchef)
6. [Homepage](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Homepage)
7. [*Netbox](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Netbox)
8. [Drawio](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Drawio)
9. [Actual Budget for financial budgeting](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/ActualBudget)
10. [Stirling PDF for PDF Tools](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Stirling-PDF)
11. [FireflyIII](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/FireflyIII)
12. [Collabora online](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Collabora)
13. [Nextcloud](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Nextcloud)
14. [Media Server: Transmission, Sonarr, Radarr, Readarr, Lidarr, Bazarr, Unpackerr, Prowlarr, Plex, Overseerr](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Media-Server)
15. [Calibre server and Calibre web server](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Media-Server)
16. [BOINC shared computing](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Computing/Boinc)
17. [Bookstack](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Bookstack)
18. [Vaultwarden](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Vaultwarden)
19. [OpenProject](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Openproject)
20. [Shlink URL shortener](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Shlink)
21. [Privatebin](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Privatebin)
22. [Local AI](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/LocalAI)
23. [Composecraft](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Composecraft)
24. [Paperless NGX](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Paperless-ngx)
25. [Obsidian (to do)](https://mariushosting.com/how-to-install-obsidian-on-your-synology-nas/)
26. [Github Desktop (to do)](https://mariushosting.com/how-to-install-github-desktop-on-your-synology-nas/)
27. [QRcode Generator](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/QRCodeGenerator)
28. [Reactive Resume (to do)](https://mariushosting.com/how-to-install-reactive-resume-v4-on-your-synology-nas/)
29. [RSS (to do )](https://mariushosting.com/how-to-install-rss-on-your-synology-nas/)
30. [FreshRSS](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/FreshRSS)
31. [Karakeep bookmark organizer](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Karakeep)
32. [Immich (to do)](https://mariushosting.com/how-to-install-immich-on-your-synology-nas/)
