# Proxmox

## Move disks to new storage

```bash
for vmid in {102..120};do qm move-disk $vmid scsi0 local-zfs --delete;done
```

## From 3 to 1 nodes

Stop the nodes, then:

```bash
pvecm expected 1
```

## From 1 to 3 nodes

```bash
pvecm expected 3
```

Then start the nodes.
