# ProxmoxHomeLabNotes

## Appunti vari sul progetto Proxmox

Il progetto Proxmox prevede di impostare un cluster di almeno tre nodi Proxmox, per far girare le seguenti istanze:

Plesk (mailserver)

- [ ] How to manage the mailserver with cloudflare

Proxmox (Hypervisor)

- [x] PfSense in HA
- [x] HomeAssistant (domotica)
- [x] 3x TrueNAS SCALE cluster (filesystem) - NOT WORKING
- [x] Truecommand to manage TrueNAS SCALE gluster cluster - NOT WORKING
- [x] Gluseterfs cluster
- [x] 3x haproxy for HA implementations
- [x] 3x etcd cluster for kubernetes
- [x] 3x control planes, 3x worker nodes kubernetes cluster
  - [x] Metallb Loadbalancer
  - [x] Multus multinetwork
  - [x] Longhorn storage
  - [x] Portainer
  - [x] Certmanager
  - [x] Traefik
  - [x] Proxmox monitoring (Prometheus-Grafana)
  - [x] Proxmox Backup monitoring (Prometheus-Grafana)
  - [x] Haproxy monitoring
  - [x] Dell Idrac Monitoring
  - [x] Arista switch monitoring
  - [x] Ceph monitoring
  - [x] Glusterfs monitoring
  - [x] NFS monitoring
  - [x] Uptimekuma
  - [x] Kured
  - [x] Heimdall
  - [x] Homer
  - [x] Homepage
  - [ ] Datree (deprecated)
  - [x] Teleport
  - [x] Nvidia GPU plugin
  - [x] NFS via NFS subdir provisioner
  - [x] node feature discovery (not needed with Nvidia GPU plugin)
  - [x] reloader (?)
  - [x] k8TZ (to set a default timezone for the cluster)
  - [ ] Kyverno
  - [ ] Cilium+Hubble (?)
  - [ ] Authelia+lldap
  - [ ] CloudnativePG
  - [ ] kubelet-csr-approver
  - [ ] Livesync for Obsidian (through couchdb cluster)
  - [x] Postgresql cluster with pgadmin and monitoring
  - [x] MySQL replica cluster with Proxysql and Phpmyadmin; MySQL Prometheus-Grafana monitoring
  - [x] Mariadb replica cluster with Proxysql and Phpmyadmin; Mariadb Prometheus-Grafana monitoring
  - [x] Redis and Redis Prometheus-Grafana monitoring
  - [x] Dbgate for multi-db client
  - [x] Cloudbeaver for multi-db client
  - [x] Memcached
  - [x] MinIO for S3 ObjectStorage
  - [x] ArgoCD for CD/DI
  - [x] Gitlab for CD/CI
  - [x] Hasicorp vault and External secrets for secret SPoT
  - [x] Velero Backup
  - [x] NextCloud (FileServer) and Prometheus-Grafana monitoring
    - [x] Nextcloud apps
    - [ ] Other nextcloud apps <https://www.tecmint.com/nextcloud-apps/amp/>
  - [x] Plex (MediaServer)
    - [x] With Sonarr, Radarr, Transmission, Bazarr, Lidarr, Readarr, Prowlarr, Unpackerr, Overseer
    - [ ] Tautulli, all other "rr": autobrr, omegabrr, plex-auto-languages, plex-meta-manager, recyclarr, sabnzbd, wizarr
  - [ ] Boing shared computing
  - [ ] Vaultwarden <https://github.com/dani-garcia/vaultwarden/wiki/Kubernetes-deployment>
  - [ ] the lounge
  - [x] actualbudget
  - [ ] Authelia? <https://www.authelia.com/integration/kubernetes/introduction/>
  - [ ] Guacamole? <https://github.com/thomas-illiet/k8s-guacamole>
  - [ ] UrBackup (Backup)
  - [ ] miniflux
  - [ ] shlink
  - [ ] nut management
  - [ ] paperless
  - [x] Stirling PDF
  - [ ] Reactive Resume
  - [ ] Firefly III
  - [ ] Flatnotes
  - [x] IT-tools
  - [x] Drawio

<https://github.com/awesome-selfhosted/awesome-selfhosted>