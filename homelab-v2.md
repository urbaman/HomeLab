# Homelab Architecture Design
> Documento tecnico di progettazione — Marzo 2026

---

## 1. Scelta della Distribuzione Kubernetes

### Confronto Distro

| Criterio | k3s | Talos Linux | MicroK8s |
|---|---|---|---|
| Facilità installazione | ★★★★★ | ★★★ | ★★★★ |
| Footprint risorse | Minimo | Minimo | Medio |
| Sicurezza | Buona | Ottima (immutable OS) | Buona |
| HA (etcd embedded) | ✅ | ✅ | ✅ |
| Gestione centralizzata | Rancher/Portainer | **Sidero Omni** | |
| GPU passthrough/plugin | ✅ | ✅ | ✅ |
| Adatto homelab prod | ✅ | ✅✅ | ⚠️ |
| Aggiornamenti atomici | ❌ | ✅ (rollback) | ❌ |
| Curva di apprendimento | Bassa | Media-Alta | Bassa |

### Raccomandazione

| Cluster | Distro | Motivazione |
|---|---|---|
| **Produzione** | **Talos Linux** | OS immutabile, no SSH, API-driven. Compatibile con **Sidero Omni** (già in lista) per gestione centralizzata. Perfetto per workload critici. |
| **Management** | **k3s** | Leggerissimo, facile da gestire, perfetto per tooling ops (Authentik, Gitea, ecc.). Nessun overhead Talos per cluster non prod. |
| **Dev/Test** | **k3s** | Spin-up e teardown rapidi. Cluster usa embedded SQLite (no etcd per dev). |

> **Nota su MicroK8s**: sconsigliato per cluster multi-nodo in homelab. Ottimo solo per singolo nodo di sviluppo locale.

---

## 2. Struttura VLAN

### Schema VLAN Completo

| VLAN ID | Nome | Subnet | Gateway | Scopo |
|---|---|---|---|---|
| **1** | MGMT-INFRA | 10.0.1.0/24 | 10.0.1.1 | OOB management: switch management, IPMI, console pfSense |
| **10** | LAN-TRUST | 10.0.10.0/24 | 10.0.10.1 | Rete LAN domestica/ufficio trusted (PC, laptop) |
| **20** | PROXMOX | 10.0.20.0/24 | 10.0.20.1 | Proxmox WebUI, Corosync heartbeat, migrazioni live VM |
| **30** | K8S-PROD | 10.0.30.0/24 | 10.0.30.1 | Nodi Talos cluster produzione (CP + workers) |
| **31** | K8S-PROD-LB | 10.0.31.0/28 | — | MetalLB pool produzione (.10–.30): IP virtuali servizi |
| **40** | K8S-MGMT | 10.0.40.0/24 | 10.0.40.1 | Nodi k3s cluster management |
| **41** | K8S-MGMT-LB | 10.0.41.0/28 | — | MetalLB pool management (.10–.20) |
| **50** | K8S-DEV | 10.0.50.0/24 | 10.0.50.1 | Nodi k3s cluster dev/test |
| **60** | STORAGE | 10.0.60.0/24 | 10.0.60.1 | Traffico dedicato iSCSI e NFS — completamente isolato |
| **70** | DMZ | 10.0.70.0/24 | 10.0.70.1 | Servizi esposti: Traefik ingress, reverse proxy pubblico |
| **80** | IOT | 10.0.80.0/24 | 10.0.80.1 | Dispositivi IoT, smart home (opzionale) |
| **90** | GUEST | 10.0.90.0/24 | 10.0.90.1 | WiFi guest, rete non fidata (opzionale) |

### Topologia Switch

```
Internet
    │
  pfSense (Qotom Q20300S9)
    │ (trunk all VLANs)
    │
  ┌─┴────────────────────────────┐
  │  Switch Core: 12x10G SFP+   │  ← backbone 10G
  └──┬──────┬───────┬────────────┘
     │      │       │
   DAC    DAC     DAC/SFP+
     │      │       │
  MS-01  NAS      Switch Access
(10G SFP+) (FlashStation) (8x2.5G+2x10G+2x10SFP+)
                              │
                    ┌─────────┼──────────┐
                  M920q    HP 800-G2   Geekom x3
                (2.5G)    (2.5G)      (2.5G)
                              │
                    Switch Small (4x2.5G+2x10SFP+)
                    → LAN trust, IoT, Guest
```

**Collegamento NAS (storage VLAN 60):**
- FlashStation FS → 12-port core switch via SFP+ 10G (iSCSI + NFS su VLAN 60)
- MS-01 → 12-port core switch via SFP+ 10G (storage NIC dedicata su VLAN 60)
- Tutti gli altri nodi → 8-port switch via 2.5G (VLAN 60 in trunk, sufficiente per DB headless)

### Regole Firewall pfSense (Policy principali)

| Sorgente | Destinazione | Policy | Note |
|---|---|---|---|
| LAN-TRUST | MGMT-INFRA | Allow (gestione solo da trusted) | |
| LAN-TRUST | PROXMOX | Allow porta 8006 | WebUI Proxmox |
| LAN-TRUST | K8S-PROD-LB | Allow HTTPS (443) | Accesso app prod |
| LAN-TRUST | K8S-MGMT-LB | Allow HTTPS (443) | Accesso tool mgmt |
| PROXMOX | STORAGE | Allow NFS/iSCSI | Solo porte 2049, 3260 |
| K8S-PROD | STORAGE | Allow NFS/iSCSI | Solo porte 2049, 3260 |
| K8S-MGMT | STORAGE | Allow NFS/iSCSI | Solo porte 2049, 3260 |
| K8S-DEV | STORAGE | Allow NFS/iSCSI | Solo porte 2049, 3260 |
| K8S-* → K8S-* | inter-cluster | Block (default) | Eccetto chiamate API specifiche |
| WAN | DMZ | Allow 80, 443 | Solo traffico web |
| IOT | * | Block (default) | Isolato |
| GUEST | * | Block (default) | Solo internet |
| STORAGE | tutto | Block totale | Nessun routing in uscita |

---

## 3. Espansione e Allocazione Proxmox Cluster

### Cluster a 6 Nodi (aggiungere i 3x Geekom IT15)

| Nodo | CPU | RAM | Storage locale | NIC | Note |
|---|---|---|---|---|---|
| MS-01 | i9-13900 (24C/32T) | 96GB | 2x NVMe (RAID?) | 2x10G SFP+ + 2x2.5G | Nodo principale, GPU T1000 |
| M920q | i9 | 64GB | NVMe | 1x2.5G + USB-2.5G | GPU T1000 |
| HP 800-G2 | i7 | 64GB | NVMe | 1x1G + USB-2.5G | Nodo general-purpose |
| Geekom IT15 #1 | Ultra 9-285H (24C) | 32GB → **64GB** | NVMe | 2x2.5G | Upgrade RAM consigliato |
| Geekom IT15 #2 | Ultra 9-285H (24C) | 32GB → **64GB** | NVMe | 2x2.5G | Upgrade RAM consigliato |
| Geekom IT15 #3 | Ultra 9-285H (24C) | 32GB → **64GB** | NVMe | 2x2.5G | Upgrade RAM consigliato |

> **Totale cluster:** ~150 vCPU, ~352GB RAM (post upgrade)

### Allocazione VM per Nodo

#### MS-01 (96GB)
| VM | vCPU | RAM | Disco | VLAN |
|---|---|---|---|---|
| talos-prod-worker-1 | 10 | 32GB | 120GB | 30 + 60 |
| Ubuntu Desktop mgmt | 4 | 8GB | 60GB | 20 |
| *Riserva* | — | ~56GB | — | — |

> GPU T1000: passthrough diretto a talos-prod-worker-1 per LocalAI/OpenWebUI

#### M920q (64GB)
| VM | vCPU | RAM | Disco | VLAN |
|---|---|---|---|---|
| talos-prod-worker-2 | 8 | 24GB | 120GB | 30 + 60 |
| Virtualmin | 4 | 8GB | 100GB | 70 |
| *Riserva* | — | ~32GB | — | — |

> GPU T1000: passthrough a talos-prod-worker-2 (fallback AI o GPU compute)

#### HP 800-G2 (64GB)
| VM/CT | vCPU | RAM | Disco | VLAN |
|---|---|---|---|---|
| talos-prod-cp-1 | 2 | 4GB | 30GB | 30 |
| k3s-mgmt-cp-1 | 4 | 8GB | 50GB | 40 |
| CT: Proxmox Mail Gateway | 2 | 4GB | 30GB | 20 |
| CT: Proxmox Backup Server | 2 | 6GB | 40GB | 20 |
| CT: Proxmox DC Manager | 2 | 4GB | 20GB | 20 |
| *Riserva* | — | ~38GB | — | — |

#### Geekom IT15 #1 (32GB → 64GB)
| VM | vCPU | RAM | Disco | VLAN |
|---|---|---|---|---|
| talos-prod-cp-2 | 2 | 4GB | 30GB | 30 |
| k3s-mgmt-worker-1 | 6 | 16GB | 80GB | 40 |
| k3s-dev-cp-1 | 2 | 4GB | 30GB | 50 |
| *Riserva* | — | ~40GB | — | — |

#### Geekom IT15 #2 (32GB → 64GB)
| VM | vCPU | RAM | Disco | VLAN |
|---|---|---|---|---|
| talos-prod-cp-3 | 2 | 4GB | 30GB | 30 |
| k3s-mgmt-worker-2 | 6 | 16GB | 80GB | 40 |
| k3s-dev-worker-1 | 4 | 8GB | 60GB | 50 |
| *Riserva* | — | ~36GB | — | — |

#### Geekom IT15 #3 (32GB → 64GB)
| VM | vCPU | RAM | Disco | VLAN |
|---|---|---|---|---|
| talos-prod-worker-3 | 8 | 20GB | 100GB | 30 + 60 |
| k3s-dev-worker-2 | 4 | 8GB | 60GB | 50 |
| *Riserva* | — | ~36GB | — | — |

---

## 4. Storage Architecture — Flashtor12 Pro

### Setup NAS (Synology FlashStation)

Il FlashStation supporta NFS v4.1 e iSCSI su interfaccia 10G SFP+. Connettere alla **VLAN 60 (STORAGE)** tramite il core switch 12-port.

#### Dataset/Export NFS
```
/volume1/
├── k8s-prod-nfs/        → NFS export per K8s prod (RWX)
│   ├── immich/
│   ├── paperless/
│   ├── nextcloud/
│   ├── media/           → Jellyfin, arr stack
│   └── shared/
├── k8s-mgmt-nfs/        → NFS export per K8s management
├── backups/             → Proxmox Backup Server target
└── snapshots/           → Snapshot btrfs automatici
```

#### iSCSI Target Groups
```
Target Group: k8s-prod-db
  LUN-01: postgresql-prod     (200GB) → CloudNativePG
  LUN-02: mariadb-prod        (100GB) → MariaDB Operator
  LUN-03: mongodb-prod        (150GB) → MongoDB Community Op.
  LUN-04: valkey-prod          (20GB) → StatefulSet

Target Group: k8s-mgmt-db
  LUN-01: postgresql-mgmt     (50GB)  → Authentik, Gitea, ecc.
  LUN-02: valkey-mgmt          (10GB)

Target Group: k8s-dev-db
  LUN-01: dev-general         (100GB) → Pool generico dev
```

### Integrazione Kubernetes — democratic-csi

**democratic-csi** è il driver consigliato: supporta sia NFS che iSCSI con provisioning dinamico (PVC automatici).

#### Installazione (Helm)
```bash
# Aggiungere il repo
helm repo add democratic-csi https://democratic-csi.github.io/charts/

# Installare driver NFS
helm install zfs-nfs democratic-csi/democratic-csi \
  --values values-nfs.yaml \
  -n democratic-csi --create-namespace

# Installare driver iSCSI
helm install zfs-iscsi democratic-csi/democratic-csi \
  --values values-iscsi.yaml \
  -n democratic-csi
```

#### StorageClass per workload

| StorageClass | Protocollo | AccessMode | Workload tipici |
|---|---|---|---|
| `nas-iscsi-retain` | iSCSI | ReadWriteOnce | DB (PostgreSQL, MariaDB, MongoDB) |
| `nas-nfs-rwx` | NFS v4.1 | ReadWriteMany | Media, documenti, Nextcloud, Immich |
| `nas-nfs-rwo` | NFS v4.1 | ReadWriteOnce | App con singolo writer |
| `local-path` | LocalPath | ReadWriteOnce | Cache, sessioni temporanee |

#### Matrice storage per applicazione

| App | StorageClass | Volume | Note |
|---|---|---|---|
| PostgreSQL (CloudNativePG) | nas-iscsi-retain | 200GB | Operator gestisce replica |
| MariaDB | nas-iscsi-retain | 100GB | |
| MongoDB | nas-iscsi-retain | 150GB | |
| Valkey | nas-iscsi-retain | 20GB | Persist AOF |
| Memcached | EmptyDir | — | Cache volatile, no persist |
| Immich | nas-nfs-rwx | 500GB+ | Foto, video — shared access |
| Paperless-NGX | nas-nfs-rwo | 100GB | Documenti scansionati |
| Nextcloud/OpenCloud | nas-nfs-rwx | 500GB+ | Files condivisi |
| Jellyfin | nas-nfs-rwx | bind media/ | Solo lettura libreria |
| arr* stack | nas-nfs-rwx | bind media/ | Lettura/scrittura torrent/files |
| BookStack | nas-iscsi-retain | 20GB | DB interno |
| Vaultwarden | nas-iscsi-retain | 5GB | SQLite persistente |
| FreshRSS | nas-iscsi-retain | 10GB | |
| Gitea | nas-iscsi-retain | 50GB | Repo git |
| Garage S3 | nas-nfs-rwo | 200GB+ | Object storage distribuito |

### Backup Strategy

```
NAS (Flashtor12)
  └── Snapshot btrfs automatici (ogni 4h, retention 7 giorni)
      └── PBS (Proxmox Backup Server) → target NAS /backups
          ├── Backup VM K8s nodes (giornaliero, retention 14gg)
          └── Backup via velero (cluster state + PVC)

Velero (in-cluster):
  - Schedule: ogni notte
  - Target: Garage S3 (object storage interno)
  - Include: tutti i namespace, PVC via Restic
```

---

## 5. Distribuzione Applicazioni per Cluster

### Cluster Produzione — Talos Linux

#### Namespace: `databases`
| App | Metodo deploy | Operator/Helm |
|---|---|---|
| PostgreSQL | CloudNativePG Operator | `cloudnative-pg` |
| MariaDB | MariaDB Operator | `mariadb-operator` |
| MongoDB | Community Operator | `community-operator` |
| Valkey | Helm StatefulSet | `valkey` chart |
| Memcached | Helm Deployment | `memcached` chart |
| PgAdmin | Helm | `pgadmin4` |
| CloudBeaver | Helm/Kustomize | |

#### Namespace: `media`
| App | Note |
|---|---|
| Jellyfin | GPU passthrough worker-1 (T1000) |
| Radarr, Sonarr, Prowlarr, Lidarr | arr stack |
| Immich | Con NFS per storage foto |
| FreshRSS | |

#### Namespace: `productivity`
| App | Note |
|---|---|
| Nextcloud / OpenCloud | Helm chart, NFS backend |
| BookStack | |
| Paperless-NGX | OCR abilitato |
| Trilium | |
| Karakeep | |
| DrawIO | Self-hosted |
| Excalidraw | |
| OpenProject / Vikunja | Scegliere uno |
| Actual Budget | |
| Firefly III | |
| RoundCube | Con SMTP/IMAP → PMG |

#### Namespace: `tools`
| App | Note |
|---|---|
| IT-tools | |
| CyberChef | |
| Omni-tools | |
| Stirling PDF | |
| PrivateBin | |
| Shlink | URL shortener |
| QR Generator | |

#### Namespace: `ai`
| App | Note |
|---|---|
| LocalAI | Worker-1 (GPU T1000) |
| OpenWebUI | Frontend per LocalAI |
| n8n | Automazione, integra con AI |

#### Namespace: `infra` (infrastruttura cluster)
| App | Note |
|---|---|
| Traefik | IngressController, MetalLB LB |
| CrowdSec | WAF + IPS, integrazione Traefik |
| Cert-Manager | TLS automatico (Let's Encrypt / interno) |
| MetalLB | L2/BGP load balancer |
| democratic-csi | Storage provisioner |
| Dozzle | Log viewer container |
| Pulse | Monitoring |
| Vaultwarden | Password manager |
| Speedtest Tracker | |
| Teleport | Accesso sicuro SSH/K8s |

---

### Cluster Management — k3s

#### Namespace: `databases`
| App | Note |
|---|---|
| PostgreSQL | Singola istanza (o CloudNativePG) |
| Valkey | Cache per Authentik, Gitea |

#### Namespace: `gitops`
| App | Note |
|---|---|
| Gitea | SCM leggero, alternativa a GitLab |
| Semaphore UI | Ansible/Terraform runner |
| FluxCD o ArgoCD | GitOps per tutti i cluster |

#### Namespace: `identity`
| App | Note |
|---|---|
| Authentik | SSO/OIDC per tutte le app |
| OpenBao | Secrets management (fork OSS di Vault) |

#### Namespace: `infra-mgmt`
| App | Note |
|---|---|
| Sidero Omni | Gestione cluster Talos |
| Portainer | Visualizzazione container (opzionale con Sidero) |
| NetBox | IPAM e documentazione infra |
| Garage S3 | Object storage distribuito interno |
| Rackpeek / Scanopy | Inventory fisico |
| Traefik | Ingress cluster management |
| Authentik (outpost) | Proxy per proteggere tool non-OIDC |

> **Nota su Rancher vs Sidero Omni**: dato che usi Talos per il cluster prod, Sidero Omni è la scelta naturale e superiore. Rancher ha senso se gestisci cluster eterogenei (EKS, GKE, ecc.). Per homelab Talos-first: **usa Sidero Omni**.

---

### Cluster Dev/Test — k3s

- **Ambiente usa e getta**: CI/CD pipeline da Gitea → deploy su dev
- **FluxCD**: sync automatico da branch `dev` del repo
- **Risorse limitate**: 10 vCPU, 20GB RAM totali
- Storage: solo NFS (nessun iSCSI dedicato, usa namespace dev sul pool dev)
- Nessun ingress pubblico (solo accesso interno via VLAN 50)

---

## 6. Servizi Esposti (Ingress & SSL)

### Architettura Ingress

```
Internet → pfSense (NAT 80/443) → DMZ VLAN 70
    → Traefik (MetalLB IP in VLAN 31)
        ├── *.dominio.tld → app produzione
        ├── tools.dominio.tld → toolbox
        └── CrowdSec middleware (WAF)

LAN Trust (VLAN 10) → Traefik management (VLAN 41)
    ├── gitea.interno → Gitea
    ├── authentik.interno → Authentik
    └── omni.interno → Sidero Omni
```

### TLS Strategy
- **Cert-Manager** con Let's Encrypt (wildcard via DNS challenge)
- Alternativa interna: **Step CA** (PKI interna per servizi privati)
- Authentik come OIDC provider → SSO per tutte le app compatibili

---

## 7. BOM — Espansione Hardware

### Priorità Alta (necessario)

| Componente | Qtà | Motivazione |
|---|---|---|
| **RAM upgrade Geekom IT15**: DDR5 SO-DIMM 32GB (kit da 2x16GB o 1x32GB) | 3 kit | Portare da 32→64GB ciascuno. Attualmente insufficiente per ospitare 3 VM simultanee. |
| **SFP+ DAC 10G 0.5m** (Direct Attach Copper) | 6 | Core switch → MS-01, NAS, switch access 1, switch access 2, pfSense (se ha SFP+), spare |
| **NVMe 1-2TB per Geekom IT15** | 3 | Storage locale per VM disk dei nodi (se non già presenti con capacità sufficiente). Consigliato: Samsung 990 Pro 2TB |

### Priorità Media (consigliato)

| Componente | Qtà | Motivazione |
|---|---|---|
| **UPS** rack o tower (1000-1500VA) | 1-2 | Protezione apparati core: switch, NAS, pfSense, almeno 1 nodo Proxmox. pfSense integra NUT. |
| **Patch panel 24 porte Cat6A** | 1 | Organizzazione cablaggio se in rack |
| **USB-C/TB4 to 2.5G NIC** per Geekom IT15 | 3 | Seconda NIC per VLAN trunking (storage VLAN 60 dedicata). Alternativa: VLAN tagging sulla NIC primaria (funziona ma condivide banda) |

### Priorità Bassa (nice to have)

| Componente | Qtà | Motivazione |
|---|---|---|
| **SFP+ 10GBase-T transceiver** | 2-4 | Se vuoi portare nodi RJ45 sulla backbone 10G (costosi: ~40€ cad, richiedono RJ45 CAT6A) |
| **Raspberry Pi 4/5** o mini PC low-power | 1 | Nodo OOB management sempre acceso (PiKVM, Wake-on-LAN, console seriale) |
| **Espansione NAS** (se Flashtor12 non ha abbastanza capacità) | — | Dipende dall'utilizzo effettivo |

---

## 8. Roadmap di Implementazione

### Fase 1 — Fondamenta rete (settimana 1-2)
- [ ] Configurare tutte le VLAN su pfSense e switch
- [ ] Collegare switch core tramite DAC 10G
- [ ] Connettere NAS al core switch via SFP+
- [ ] Testare raggiungibilità inter-VLAN con ACL base
- [ ] Configurare NFS export e iSCSI target sul NAS

### Fase 2 — Espansione Proxmox (settimana 2-3)
- [ ] Upgrade RAM Geekom IT15 (x3)
- [ ] Aggiungere IT15 al cluster Proxmox esistente
- [ ] Configurare storage NFS/iSCSI su Proxmox (per backup PBS)
- [ ] Configurare VLAN trunk su tutte le NIC Proxmox

### Fase 3 — Cluster Management k3s (settimana 3-4)
- [ ] Deploy 3 VM per k3s management (CP + 2 workers)
- [ ] Installare k3s HA con embedded etcd
- [ ] Deploy democratic-csi (NFS + iSCSI)
- [ ] Deploy MetalLB + Traefik
- [ ] Deploy Authentik (SSO)
- [ ] Deploy Gitea + FluxCD (GitOps)
- [ ] Deploy OpenBao (secrets)
- [ ] Deploy NetBox (IPAM)

### Fase 4 — Cluster Produzione Talos (settimana 4-6)
- [ ] Deploy Sidero Omni (nel cluster mgmt)
- [ ] Creare 6 VM Talos (3 CP + 3 workers)
- [ ] Provisionare cluster via Sidero Omni
- [ ] Deploy democratic-csi su cluster prod
- [ ] Deploy MetalLB + Traefik + CrowdSec
- [ ] Deploy Cert-Manager
- [ ] Deploy database stack (PostgreSQL, MariaDB, MongoDB)
- [ ] Deploy app produzione in ordine di priorità

### Fase 5 — Cluster Dev/Test (settimana 6-7)
- [ ] Deploy k3s dev (3 VM leggere)
- [ ] Configurare pipeline CI/CD da Gitea
- [ ] Configurare FluxCD per ambiente dev

### Fase 6 — GPU & AI (settimana 7-8)
- [ ] Configurare GPU passthrough T1000 su MS-01 e M920q
- [ ] Deploy NVIDIA device plugin su Talos worker-1
- [ ] Deploy LocalAI + OpenWebUI
- [ ] Deploy n8n con integrazione AI

---

## 9. Note Tecniche Importanti

### Talos Linux — Dettagli

```yaml
# machine config — aggiungere alla sezione machine.network
# Ogni nodo Talos deve avere NIC in VLAN 30 (K8s) e VLAN 60 (storage)
machine:
  network:
    interfaces:
      - interface: eth0
        vip:
          ip: 10.0.30.50  # VIP control plane
      - interface: eth1
        addresses:
          - 10.0.60.x/24   # storage NIC dedicata
```

### k3s HA Setup

```bash
# Primo server (embedded etcd)
curl -sfL https://get.k3s.io | sh -s - server \
  --cluster-init \
  --disable traefik \           # useremo traefik via Helm
  --disable servicelb \         # useremo MetalLB
  --node-taint "node-role.kubernetes.io/control-plane=:NoSchedule" \
  --tls-san 10.0.40.10 \        # VIP MetalLB mgmt
  --flannel-iface eth0

# Nodi aggiuntivi
curl -sfL https://get.k3s.io | sh -s - server \
  --server https://10.0.40.10:6443 \
  --token <TOKEN>
```

### democratic-csi — Config NFS (esempio)

```yaml
# values-nfs.yaml
csiDriver:
  name: "org.democratic-csi.nfs"

driver:
  config:
    driver: freenas-nfs  # oppure synology-nfs
    instance_id: "flashtor12"
    httpConnection:
      host: 10.0.60.10   # IP NAS su VLAN storage
      port: 5000
      apiKey: "xxxxx"
    nfs:
      shareHost: 10.0.60.10
      shareAlldirs: false
      shareAllowedHosts: []
      shareAllowedNetworks: ["10.0.30.0/24","10.0.40.0/24","10.0.50.0/24"]
```

### GPU Passthrough Proxmox → Talos

```bash
# In /etc/modprobe.d/pve-blacklist.conf su nodo Proxmox
blacklist nouveau
blacklist nvidia
options vfio-pci ids=10de:1cb3  # PCI ID del T1000

# Nel config VM Talos (tramite Proxmox UI)
# Machine: q35
# BIOS: OVMF (UEFI)
# Aggiungere PCI device: GPU T1000, passthrough=1, rombar=0
```

```yaml
# Nei Talos machine config, abilitare iommu kernel args
machine:
  install:
    extraKernelArgs:
      - "intel_iommu=on"
      - "iommu=pt"
```
