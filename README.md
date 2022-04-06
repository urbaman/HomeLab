# ProxmoxHomeLabNotes
## Appunti vari sul progetto Proxmox
Il progetto Proxmox prevede di impostare un cluster di almeno tre nodi Proxmox, per far girare le seguenti istanze:

Proxmox (Hypervisor)
- NextCloud (FileServer)
- Plex (MediaServer)
- UrBackup (Backup)
- HomeAssistant (domotica)
- Bitwarden (Passmanager)
- PiHole (Firewall)
- TrueNAS (filesystem)
- HEIMDALL (AppDashboard)
- debian(s) con Docker Swarm o Microk8s
    (https://thehomelab.wiki/books/dns-reverse-proxy)
    - Nginx proxy manager
	- Cloudflare DNS docker
	- MariaDB docker
	- Authelia
	- PiHole
      - Guacamole

Server specs:
- Disco(i) per OS (3x250GB - raidz)
- Disco(i) per VMs (3x1 tera - raidz)
- Disco(i) per TrueNas con HBA in IT mode (3x4 tera - raidz)

Appunti per how-to da seguire:
- Flash H310: https://fohdeesha.com/docs/H310.html
- Generale: https://www.kreaweb.be/diy-home-server-introduction/
- GPU distribuita: https://drive.google.com/drive/folders/15hGRYQGB69pES2GyUR9Q2zHVvEAVHxDa
- Post-install: https://opensourcelibs.com/lib/xshok-proxmox, https://opensourcelibs.com/lib/proxmox-zfs-postinstall
- Risorse vari: https://opensourcelibs.com/lib/awesome-proxmox-ve
- Netdata: https://community.netdata.cloud/t/best-way-to-setup-netdata-in-a-proxmox-host-with-some-lxc-containers/81
- Ceph: https://blog.miniserver.it/proxmox/come-fare-un-cluster-proxmox-con-ceph/

ZFS: aggiungere un disco creando un mirror:

`#backup the partition table from da0 to restore to (new) da1 
gpart backup vtbd0 > /tmp/vtbd0.bak
gpart restore -l vtbd1 < /tmp/vtbd0.bak

#add new partitioned disk to zpool
zpool attach pfSense vtbd0p3 vtbd1p3

#write bootloader to new disk
gpart bootcode -b /boot/pmbr -p /boot/gptzfsboot -i 2 vtbd1

#write EFI partition to new disk
dd if=/boot/boot1.efifat of=/dev/vtbd1p1 bs=4m
`
