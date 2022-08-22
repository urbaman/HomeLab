# Proxmox

## Move disks to new storage

```bash
for vmid in {102..120};do qm move-disk $vmid scsi0 local-zfs --delete;done
```

## Managing the summer-winter heat/consume

### From 3 to 1 nodes

On all nodes:

```bash
systemctl stop pve-cluster
systemctl stop corosync
```

Start the cluster filesystem again in local mode:

```bash
pmxcfs -l
```

Delete the corosync configuration files:

```bash
rm /etc/pve/corosync.conf
rm /etc/corosync/*
```

You can now start the filesystem again as normal service:

```bash
killall pmxcfs
systemctl start pve-cluster
```

Stop the nodes.
