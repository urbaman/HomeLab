### ZFS: aggiungere un disco creando un mirror:

psSense

```
#check the zpool and the available drives (freeBSD/pfSense)
zpool status
sysctl kern.disks

#backup the partition table from da0 to restore to (new) da1 
gpart backup da0 > /tmp/da0.bak
gpart restore -l da1 < /tmp/da0.bak

#add new partitioned disk to zpool zpool1
zpool attach zpool1 da0p3 da1p3

#write bootloader to new disk
gpart bootcode -b /boot/pmbr -p /boot/gptzfsboot -i 2 da1

#write EFI partition to new disk
dd if=/boot/boot1.efifat of=/dev/da1p1 bs=4m
```

TrueNAS
```
gdisk /dev/sda
  b (enter filename)

#5: restore partition backup to sdb with gdisk
 gdisk /dev/sdb
r (recover and then l to load backup filename

#6: Check the boot-pool status
It should display boot-pool and the largest partition of the sda drive like following
 zpool status boot-pool
  pool: boot-pool
state: ONLINE
config:

    NAME        STATE     READ WRITE CKSUM
    boot-pool  ONLINE       0     0     0
    sda3       ONLINE       0     0     0

errors: No known data errors

#7: Attach the sdb to the zpool
Keep in mind to use the largest partition which should be the same as on sda
 zpool attach boot-pool sdb3

#8: Copy the EFI Boot partition
 dd if=/dev/sda2 of=/dev/sdb2

#9: Copy the BIOS Boot partition (not sure if needed)
 dd if=/dev/sda1 of=/dev/sdb1
 ```
