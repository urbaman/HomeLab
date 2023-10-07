# Lots of Linux Tips

## Purge previously uninstalled packages

```bash
sudo apt purge `dpkg --list | grep ^rc | awk '{ print $2; }'`
```

## list disks

```bash
ls -l /dev/disk/by-id
ls -l /dev/disk/by-uuid
sudo hwinfo --disk
sudo hwinfo --disk --short
sudo fdisk -l
sudo lshw -class disk
sudo lshw -class disk | grep <disk_name> -A 5 -B
lsblk
lsblk -f
```

## Modify text with sed

```bash
sed -i 's/old-text/new-text/g' input.txt
```
