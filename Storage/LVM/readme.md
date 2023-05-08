# LVM management

## Situation

Let's say I add two new disks to a machine, /dev/sdb and /dev/sdc.

## Create the Physical Volume(s)

```
sudo pvcreate /dev/sdb /dev/sdc
sudo pvs
sudo pvdisplay
```

## Create the Volume Group(s)

I add both disks to the same Volume Group in stripe mode, using all of the space available

```
sudo vgcreate vg1 /dev/sdb /dev/sdc
sudo vgs
sudo vgdisplay
```

## Create the Logical Volume and format it

```
sudo lvcreate -l +100%FREE -n lv1 vg1
sudo lvs
sudo lvdisplay
sudo mkfs.ext4 /dev/vg1/lv1
```

## Mount automatically

### Get the needed info on the volume

```
sudo ls -la /dev/vg1/lv1
```

Output

```
lrwxrwxrwx 1 root root 7 May  8 18:30 /dev/vg1/lv1 -> ../dm-1
```

```
sudo ls -la /dev/disk/by-id/
```

Output

```
...
lrwxrwxrwx 1 root root  10 May  8 18:30 dm-uuid-LVM-UCOJAHlWkltuQXettYxpHK4IkMzW1HAxNgp1kONtA3X5Q0mqyG5Fje9R8RjlmD11 -> ../../dm-1
...
```

Match the "/dm-1" on both outputs, keep the dm-uuid-LVM-xxxxxxxx value at hand

```
sudo mkdir /longhorn
sudo mkdir /longhorn/storage
```

```
sudo vi /etc/fstab
```

Add

```
/dev/disk/by-id/dm-uuid-LVM-UCOJAHlWkltuQXettYxpHK4IkMzW1HAxNgp1kONtA3X5Q0mqyG5Fje9R8RjlmD11 /longhorn/storage ext4 defaults 0 0
```

## Resize

### Resize one of the disks

If we need to resize one of the disks (maybe we are using a Virtual Machine), let's do:

```
growpart /dev/sdX Y #only if used a partition, e.g. /dev/sda3 goes to growpart /dev/sda 3
pvresize /dev/sdX
lvextend -l +100%FREE /dev/vg1/lv1
resize2fs /dev/vg1/lv1
```

### Add a disk

```
sudo pvcreate /dev/sdd
sudo vgextend vg1 /dev/sdd
sudo vgs -S vgname=lvm_tutorial -o vg_free
```

We should see the size of the new disk as free space in the volume

sudo lvresize -l +100%FREE vg1/lv1
sudo resize2fs /dev/vg1/lv1
