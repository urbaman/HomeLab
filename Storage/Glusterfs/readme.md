# Clustered storage - Gluster on ZFS

## On all nodes

### zpools creation

```bash
sudo apt install zfsutils-linux
```

```bash
sudo fdisk -l
```

```bash
sudo zpool create new-pool mirror /dev/sdb /dev/sdc
```

```bash
sudo zpool create HDD5T mirror /dev/sdb /dev/sdc
sudo zpool create SDD2T mirror /dev/sdd /dev/sde
```

### Zpool import

Check available pools, then import them

```bash
sudo zpool import
sudo zpool import pool1 pool2
```

### Gluster installation

```bash
sudo apt install glusterfs-server
sudo systemctl enable glusterd
sudo systemctl start glusterd
```

## On first node

### Set up gluster cluster

```bash
sudo gluster peer probe gluster2.domain.com
sudo gluster peer probe gluster3.domain.com
sudo gluster peer status
```

### Set up Gluster replicated volumes

We create two volumes. Having three nodes, for better resiliency we create distributed volumes with replica 3 (euqal to nodes number)

```bash
sudo gluster volume create HDD5T replica 3 gluster1.domain.com:/HDD5T11 gluster1.domain.com:/HDD5T12 gluster2.domain.com:/HDD5T21 gluster2.domain.com:/HDD5T22 gluster3.domain.com:/HDD5T31 gluster3.domain.com:/HDD5T32
sudo gluster volume create SDD2T replica 3 gluster1.domain.com:/SDD2T11 gluster1.domain.com:/SDD2T12 gluster2.domain.com:/SDD2T21 gluster2.domain.com:/SDD2T22 gluster3.domain.com:/SDD2T31 gluster3.domain.com:/SDD2T32
```

To check the volumes:

```bash
sudo gluster volume status
sudo gluster volume info
```

## Mount at boot with systemd mount units

Edit a file named after the mount path: `mounthpath.mount`.

>**_NOTE:_** Don’t use a dash - in your path because a dash refer to a new branch on the folder tree and this will break the naming of the mount/automount file
>
>- Invalid /data/home-backup
>- Valid /data/home_backup
>- Valid /data/home\x2dbackup
>
>To get the correct name for a mount filename when you use dashes in your folder-naming
>
>```bash
>systemd-escape -p --suffix=mount "/data/home-backup"
>data/home\x2dbackup.mount
>```
>
>For automount
>
>```bash
>systemd-escape -p --suffix=automount "/data/home-backup"
>data/home\x2dbackup.automount
>```

```bash
[Unit]
Description=Mount glusterfs volumename
After=/lib/systemd/system/glusterd.service
Wants=/lib/systemd/system/glusterd.service

[Mount]
What=thisserverfullhostname:/volumename
Where=/mountpath
Type=glusterfs
Options=_netdev,auto

[Install]
WantedBy=multi-user.target
```

Then, reload systemd daemon, enable and start the unit.

```bash
sudo systemd daemon-reload
sudo systemd enable mountpath.mount
sudo systemd start mountpath.mount
```

## Disk substitution

Substitute the disk, check the zpools and fdisk to find the disk without a pool or the pool without a disk, recreate the zpool or add the disk to the pool missing it, then reset the gluster brick if faulty:

```bash
sudo fdisk -l
sudo zpool status
sudo zpool create new-pool mirror /dev/sdb /dev/sdc
sudo gluster volume reset-brick HDD5T gluster1.domain.com:/HDD5T12 gluster1.domain.com:/HDD5T12 commit force
```
