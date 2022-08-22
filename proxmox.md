# Proxmox

## Move disks to new storage

```bash
for vmid in {102..120};do qm move-disk $vmid scsi0 local-zfs --delete;done
```

## Managing the summer-winter heat/consume

## Move VM to same node

## Move to local storage

See Move disks to new storage

## Remove Ceph

1 Remove/Delete Ceph

Warning: Removing/Deleting ceph will remove/delete all data stored on ceph as well!

1.1 Login to Proxmox Web GUI

1.2 Click on one of the PVE nodes

1.3 From right hand side panel, Navigate to Ceph -> Pools record items under Name

1.4 Navigate to Ceph -> CephFS, record existing cephFS names

1.5 From left hand side menu, Click on Datacenter

1.6 From right hand side, Click on Storage

1.7 Delete all items which we saw under CephFS and Pools from step 3 and step 4

1.8 From right hand side panel, Click on master node, Navigate to Ceph -> CephFS, Stop and Destroy all Metadata Services

1.9 Click on master node, from right hand side panel, Navigate to Ceph -> OSD, Mark all OSDs as Out

1.10 From right hand side menu, Right Click on one of the PVE nodes’ name, Click on >_ Shell to open terminal

1.11 Mark OSD down

```bash
ceph osd down <osdid>
# e.g.
ceph osd down 0
ceph osd down 1
ceph osd down 2
```

1.12 Click on master node, from right hand side panel, Navigate to Ceph -> OSD, Click on the OSD to be removed, Click on More button from top right corner, Click on Destroy

or

We can mark them down and destroy in one command

```bash
ceph osd down 0 && ceph osd destroy 0 --force
ceph osd down 1 && ceph osd destroy 1 --force
ceph osd down 2 && ceph osd destroy 2 --force
```

1.13 Remove ceph configuration file by executing the following command from terminal (Refer to step 10)

```bash
rm /etc/ceph/ceph.conf
```

1.14 On each of the PVE node, execute the following command to stop ceph monitor service

```bash
systemctl stop ceph-mon@<hostname or monid>
# e.g.
systemctl stop ceph-mon@labnode1
```

Note: If unsure what should be the hostname or monid to use after @, we can type the command “ systemctl stop ceph-mon@ ” then Press Tab key once, the correct value will be appended for us, remember to note down the appended value, we will need this value for next step (step 15)

1.15 Disable the ceph monitor service, execute the following command on each nodes

```bash
systemctl disable ceph-mon@<hostname or monid>
# e.g.
systemctl disable ceph-mon@labnode1
```

Note: Append the value we got from step 14 after “ @ “

If we see something similar to below, it means we have successfully executed the correct command

Removed /etc/systemd/system/ceph-mon.target.wants/ceph-mon@labnode1.service.

1.16 Remove ceph configuration file from all nodes

```bash
rm -r /etc/pve/ceph.conf
rm -r /etc/ceph
rm -rf /var/lib/ceph
```

If we get the following error or similar

```bash
rm: cannot remove '/var/lib/ceph/osd/ceph-0': Device or resource busy
```

We can use this command to unmonut first then, try to remove it again

```bash
umount /var/lib/ceph/osd/ceph-0
rm -r /var/lib/ceph
```

1.17 Reboot all PVE nodes

1.18 Clear leftover ceph configuration files and services, execute the following command on each nodes

```bash
pveceph purge
```

1.20 Clear the OSD disks from each OSD nodes, so that we can use those disks later

```bash
# Remove the lvm signature from the disk
# Note: Change the drive letter (sdx) accordingly
fdisk /dev/sdx
```

Then Enter d , Press Enter key, Enter w , Press Enter key

```bash
rm -r /dev/ceph-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxxx
```

Note: To find the correct value, we can use this command “ ls /dev | grep ceph ” or just type “ rm -r /dev/ceph- ” then Press Tab key to auto complete the rest

```bash
# Restart the PVE host
reboot
```

1.21 Remove all ceph packages if desired

```bash
apt purge ceph-mon ceph-osd ceph-mgr ceph-mds
```

More to remove if desired

```bash
apt purge ceph-base ceph-mgr-modules-core
```

2 Reinstall Ceph

The reinstallation procedure is almost identical with new/fresh ceph installation (really easy when using PVE web gui), however, if we have encountered the following error during creation of OSD, due to leftover from previous installation

```bash
error during cfs-locked 'file-ceph_conf' operation: command 'cp /etc/pve/priv/ceph.client.admin.keyring /etc/ceph/ceph.client.admin.keyring' failed: exit code 1 (500)
```

or

```bash
unable to open file '/var/lib/ceph/bootstrap-osd/ceph.keyring.tmp.1694' - No such file or directory (500)
```

or

```bash
unable to open file '/var/lib/ceph/bootstrap-osd/ceph.keyring.tmp.920' - No such file or directory (500)
```

To fix this issue, we can use the following method (Create the missing directory, “ /var/lib/ceph/bootstrap-osd/ “)

```bash
mkdir /var/lib/ceph/bootstrap-osd
```

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
