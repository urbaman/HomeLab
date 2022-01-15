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
- debian(s) con Docker Swarm
    (https://thehomelab.wiki/books/dns-reverse-proxy)
    - Nginx proxy manager
	- Cloudflare DNS docker
	- MariaDB docker
	- Authelia
	- PiHole

Server specs:
- Disco(i) per OS (3x250GB - raidz)
- Disco(i) per VMs (3x1 tera - raidz)
- Disco(i) per TrueNas con HBA in IT mode (3x4 tera - raidz)

Appunti per how-to da seguire:
- Generale: https://www.kreaweb.be/diy-home-server-introduction/
- GPU distribuita: 
