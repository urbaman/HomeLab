# ProxmoxHomeLabNotes
## Appunti vari sul progetto Proxmox
Il progetto Proxmox prevede di impostare un cluster di almeno tre nodi Proxmox, per far girare le seguenti istanze:

Proxmox (Hypervisor)
- [x] PfSense in HA
- [x] HomeAssistant (domotica)
- [x] 3x TrueNAS SCALE cluster (filesystem)
- [x] Truecommand to manage TrueNAS SCALE gluster cluster
- [x] 3x haproxy for HA implementations
- [x] 3x etcd cluster for kubernetes
- [x] 3x ccontrol planes, 3x worker nodes kubernetes cluster
    - [x] Metallb Loadbalancer
    - [x] Containeroo Cloudflare Operator
    - [x] Portainer
    - [x] Traefik
    - [x] Proxmox monitoring (Prometheus-Grafana)
    - [x] MySQL (MariaDB) and MySQL Prometheus-Grafana monitoring
    - [x] Redis and Redis Prometheus-Grafana monitoring
    - [x] NextCloud (FileServer) and Prometheus-Grafana monitoring
    - [ ] Plex (MediaServer) https://www.debontonline.com/2021/01/part-14-deploy-plexserver-yaml-with.html
    - [ ] Vaultwarden https://github.com/dani-garcia/vaultwarden/wiki/Kubernetes-deployment
    - [ ] Authelia? https://www.authelia.com/integration/kubernetes/introduction/
    - [ ] Guacamole? https://github.com/thomas-illiet/k8s-guacamole
    - [ ] UrBackup (Backup)
    - [ ] HEIMDALL (AppDashboard) https://www.debontonline.com/2020/12/kubernetes-part-12-deploy-heimdall-yaml.html

https://github.com/awesome-selfhosted/awesome-selfhosted

### Server specs:
- Disco(i) per OS (3x250GB - raidz)
- Disco(i) per VMs (3x1 tera - raidz)
- Disco(i) per TrueNas con HBA in IT mode (3x4 tera - raidz)

### Appunti per how-to da seguire:
- Flash H310: https://fohdeesha.com/docs/H310.html
- Generale: https://www.kreaweb.be/diy-home-server-introduction/
- GPU distribuita: https://drive.google.com/drive/folders/15hGRYQGB69pES2GyUR9Q2zHVvEAVHxDa
- Post-install: https://opensourcelibs.com/lib/xshok-proxmox, https://opensourcelibs.com/lib/proxmox-zfs-postinstall
- Risorse varie: https://opensourcelibs.com/lib/awesome-proxmox-ve
- Ceph: https://blog.miniserver.it/proxmox/come-fare-un-cluster-proxmox-con-ceph/
