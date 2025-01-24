# Kubernetes Cluser

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
2. [Metrics Server](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Metrics-Server)
3. [Cert Manager for SSL certs](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cert-manager)
4. [Storage with Glusterfs](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Storage/Glusterfs)
5. [Storage with Rook (backed by Proxmox Ceph)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Storage/Rook)
6. [Second network with Multus and Whereabouts](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Multus)
7. [Storage with longhorn](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Storage/Longhorn)
8. [Storage with NFS](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Storage/NFS)
9. [Cloudflare Operator (DynDNS, DNS - deprecated)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cloudflare-Operator)
10. [Metallb LoadBalancer](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Metallb)
11. [Kured for automatic reboots](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Kured)
12. [k8TZ](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/k8tz)
13. [Node feature discovery - not needed with Nvidia GPU operator](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Node-Feature-Discovery)
14. [Nvidia GPU operator](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Nvidia-GPU)
15. [Intel GPU Operator](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Intel-GPU)
16. [ClamAV antivirus server](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/ClamAV)
17. [Reloader (?)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Reloader)
18. [Traefik for ingress and proxy](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Traefik)
19. [Crowdsec security WAF for Traefik](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Crowdsec)
20. [Rancher dashboard](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Rancher)
21. [Monitoring with Prometheus-stack (Prometheus, Grafana)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack)
22. [Kubernetes Dashboard](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Dashboard)
23. [Portainer managing dashboard](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Portainer)
24. [Kubetail - k8s log viewer](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Kubetail)
25. [Groundcover for observability](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Groundcover)
26. [Kubescape for observability and security scan](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Kubescape)
27. [Uptime Kuma for service uptime monitoring](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Uptimekuma)
28. [Teleport for external access](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Teleport)
29. [Postgresql replica cluster with Pgadmin](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Postgresql)
30. [Mysql replica cluster with Proxysql and Phpmyadmin](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Mysql)
31. [Mariadb replica cluster with Proxysql and Phpmyadmin](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Mariadb)
32. [Redis replica cluster](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Redis)
33. [Dbgate DB web client](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Dbgate)
34. [Cloudbeaver DB web client](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Cloudbeaver)
35. [Memcached](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Memcached)
36. [MinIO for S3 object Storage](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Storage/MinIO)
37. [Monitoring Calico](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Calico)
38. [Monitoring Longhorn](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Storage/Longhorn)
39. [Monitoring External Etcd](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/ExternalEtcd)
40. [Monitoring Cert Manager](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Cert-manager)
41. [Monitoring Kured](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Kured)
42. [Monitoring Proxmox](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Proxmox-Monitoring)
43. [Monitoring Proxmox Backup Server](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Proxmox-Backup-Monitoring)
44. [Monitoring Haproxy](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Haproxy-Monitoring)
45. [Monitoring snmp (iDRAC, switch)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Prometheus-snmp)
46. [Monitoring Fritzbox](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Fritzbox-exporter)
47. [Monitoring Glusterfs](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Storage/Glusterfs)
48. [Monitoring Ceph](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Storage/Ceph)
49. [Monitoring NFS](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Storage/NFS-server)
50. [Monitoring UptimeKuma](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Uptime-kuma)
51. [Monitoring kubescape](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Kubescape)
52. [Monitoring Postgresql](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Database/Postgresql)
53. [Monitoring Mysql](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Database/Mysql)
54. [Monitoring Redis](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Database/Redis)
55. [Monitoring Memcached](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Database/Memcached)
56. [Monitoring Crowdsec](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Crowdsec)
57. [Traefik for External Service: Shared Hosting](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Hosting)
58. [Crowdsec security WAF for Traefik](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Crowdsec)
59. [Kubernetes Dashboard](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Dashboard)
60. [Portainer managing dashboard](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Portainer)
61. [Datree secure the cluster and avoid misconfigurations - deprecated](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Datree)
62. [Kubetail - k8s log viewer](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Kubetail)
63. [Groundcover for observability](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Groundcover)
64. [Kubescape for observability and security scan](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Kubescape)
65. [Uptime Kuma for service uptime monitoring](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Uptimekuma)
66. [Teleport for external access](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Teleport)
67. [Postgresql with Pgadmin](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Postgresql)
68. [Postgresql replica cluster with Pgadmin](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Postgresql-ha)
69. [Mysql standalone or cluster with Proxysql and Phpmyadmin](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Mysql)
70. [Mariadb standalone or replica cluster with Proxysql and Phpmyadmin](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Mariadb)
71. [Redis standalone or replica cluster](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Redis)
71. [Mongodb standalone or replica cluster](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Mongodb)
72. [Dbgate DB web client](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Dbgate)
73. [Cloudbeaver DB web client](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Cloudbeaver)
74. [Memcached](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Memcached)
75. [MinIO for S3 object Storage](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Storage/MinIO)
76. [Monitoring Calico](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Calico)
77. [Monitoring Longhorn](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Storage/Longhorn)
78. [Monitoring External Etcd](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/ExternalEtcd)
79. [Monitoring Cert Manager](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Cert-manager)
80. [Monitoring Kured](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Kured)
81. [Monitoring Proxmox](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Proxmox-Monitoring)
82. [Monitoring Proxmox Backup Server](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Proxmox-Backup-Monitoring)
83. [Monitoring Haproxy](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Haproxy-Monitoring)
84. [Monitoring snmp (iDRAC, switch)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Prometheus-snmp)
85. [Monitoring Fritzbox](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Fritzbox-exporter)
86. [Monitoring Glusterfs](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Storage/Glusterfs)
87. [Monitoring Ceph](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Storage/Ceph)
88. [Monitoring NFS](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Storage/NFS-server)
89. [Monitoring UptimeKuma](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Uptime-kuma)
90. [Monitoring kubescape](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Kubescape)
91. [Monitoring Postgresql](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Database/Postgresql)
92. [Monitoring Mysql](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Database/Mysql)
93. [Monitoring Redis](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Database/Redis)
93. [Monitoring Mongodb](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Database/Mongodb)
94. [Monitoring Memcached](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Database/Memcached)
95. [Monitoring Crowdsec](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack/Crowdsec)

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
23. [Obsidian (to do)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Obsidian)
24. [Github Desktop (to do)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/LocalAI)
25. [Composecraft (to do)](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Composecraft)
