# Clutered storage - Gluster on ZFS

## On all nodes

### zpools creation

```
sudo apt install zfsutils-linux
```

```
sudo fdisk -l
```

```
sudo zpool create new-pool mirror /dev/sdb /dev/sdc
```

```
sudo zpool create HDD5T mirror /dev/sdb /dev/sdc
sudo zpool create SDD2T mirror /dev/sdd /dev/sde
```

### Gluster installation

```
sudo apt install glusterfs-server
sudo systemctl enable glusterd
sudo systemctl start glusterd
```

## On first node

### Set up gluster cluster

```
sudo gluster peer probe gluster2.urbaman.it
sudo gluster peer probe gluster3.urbaman.it
sudo gluster peer status
```

### Set up Gluster replicated volumes

We create two volumes. Having three nodes, for better resiliency we create distributed volumes with replica 3 (euqal to nodes number)

```
sudo gluster volume create HDD5T replica 3 gluster1.urbaman.it:/HDD5T11 gluster1.urbaman.it:/HDD5T12 gluster2.urbaman.it:/HDD5T21 gluster2.urbaman.it:/HDD5T22 gluster3.urbaman.it:/HDD5T31 gluster3.urbaman.it:/HDD5T32
sudo gluster volume create SDD2T replica 3 gluster1.urbaman.it:/SDD2T11 gluster1.urbaman.it:/SDD2T12 gluster2.urbaman.it:/SDD2T21 gluster2.urbaman.it:/SDD2T22 gluster3.urbaman.it:/SDD2T31 gluster3.urbaman.it:/SDD2T32
```

To check the volumes:

```
sudo gluster volume status
sudo gluster volume info
```
