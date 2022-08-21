# Proxmox

## Move disks to new storage

```bash
for vmid in {102..120};do qm move-disk $vmid scsi0 local-zfs --delete;done
```
