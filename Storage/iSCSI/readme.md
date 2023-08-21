# iSCSI shared storage

## Instal and setup the Target

### Install administration tool

```bash
sudo apt -y install targetcli-fb
```

### Configure iSCSI Target

For example, Create an disk-image under the [/var/lib/iscsi_disks] directory and set it as a SCSI device.

```bash
sudo mkdir /var/lib/iscsi_disks

sudo targetcli
Warning: Could not load preferences file /root/.targetcli/prefs.bin.
targetcli shell version 2.1.53
Copyright 2011-2013 by Datera, Inc and others.
For help on commands, type 'help'.

cd backstores/fileio 
create disk01 /var/lib/iscsi_disks/disk01.img 10G 
cd /iscsi 
create iqn.2022-04.world.srv:dlp.target01 # naming convention is iqn.yyy-mm.reversed-server-hostname.targetname
Created target iqn.2022-04.world.srv:dlp.target01.
Created TPG 1.
Global pref auto_add_default_portal=true
Created default portal listening on all IPs (0.0.0.0), port 3260.
cd iqn.2022-04.world.srv:dlp.target01/tpg1/luns
create /backstores/fileio/disk01 
Created LUN 0.
cd ../acls 
create iqn.2022-04.world.srv:node01.initiator01 # This is the initiator, will need to set one for each client, then set it in the client(s)
Created Node ACL for iqn.2022-04.world.srv:node01.initiator01
Created mapped LUN 0.
cd iqn.2022-04.world.srv:node01.initiator01 
set auth userid=username 
Parameter userid is now 'username'.
set auth password=password 
Parameter password is now 'password'.
exit 
Global pref auto_save_on_exit=true
Configuration saved to /etc/rtslib-fb-target/saveconfig.json
```

Check the service is listening to port 3260:

```bash
sudo ss -napt | grep 3260
LISTEN 0      256          0.0.0.0:3260       0.0.0.0:*
```

Enable and restart the service:

```bash
sudo systemctl enable rtslib-fb-targetctl
sudo systemctl restart rtslib-fb-targetctl
```

## Install and setup the initiator(s)

### Configure iSCSI Initiator to connect to iSCSI Target

Install and set the initiator name (you already set it in the Target, or remember to add it to the acls)

```bash
sudo apt -y install open-iscsi
sudo vi /etc/iscsi/initiatorname.iscsi
```

```bash
InitiatorName=iqn.2022-04.world.srv:node01.initiator01
```

Set the authentication

```bash
sudo vi /etc/iscsi/iscsid.conf
```

```bash
node.session.auth.authmethod = CHAP
node.session.auth.username = username
node.session.auth.password = password
```

```bash
sudo systemctl restart iscsid open-iscsi
```

Discover and log in to the Target

```bash
sudo iscsiadm -m discovery -t sendtargets -p 10.0.0.30
10.0.0.30:3260,1 iqn.2022-04.world.srv:dlp.target01
sudo iscsiadm -m node -o show
# BEGIN RECORD 2.1.5
node.name = iqn.2022-04.world.srv:dlp.target01
node.tpgt = 1
node.startup = manual
node.leading_login = No
iface.iscsi_ifacename = default
.....
.....
node.conn[0].iscsi.DataDigest = None
node.conn[0].iscsi.IFMarker = No
node.conn[0].iscsi.OFMarker = No
# END RECORD

sudo iscsiadm -m node --login
Logging in to [iface: default, target: iqn.2022-04.world.srv:dlp.target01, portal: 10.0.0.30,3260]
Login to [iface: default, target: iqn.2022-04.world.srv:dlp.target01, portal: 10.0.0.30,3260] successful.

sudo iscsiadm -m session -o show
tcp: [1] 10.0.0.30:3260,1 iqn.2022-04.world.srv:dlp.target01 (non-flash)

sudo cat /proc/partitions
major minor  #blocks  name

   7        0      81868 loop0
   7        1      63380 loop1
   7        2      45748 loop2
 252        0   31457280 sda
 252        1       1024 sda1
 252        2    2097152 sda2
 252        3   29357056 sda3
 253        0   28311552 dm-0
   8        0   10485760 sdb
# added new device provided from the target server as [sdb]
```

Is using it as fencing device in a pacemaker-corosync cluster, this is enough, if you plan to use the storage directly, go on.

Only on first client

```bash
sudo parted --script /dev/sdb "mklabel gpt"
sudo parted --script /dev/sdb "mkpart primary 0% 100%"
sudo mkfs.ext4 /dev/sdb1
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 2617344 4k blocks and 655360 inodes
Filesystem UUID: 6eec40d4-a75c-4a6e-97eb-cfb545d696ee
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

sudo mount /dev/sdb1 /mnt
sudo df -hT
Filesystem                        Type   Size  Used Avail Use% Mounted on
tmpfs                             tmpfs  393M  1.1M  392M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv ext4    27G  5.7G   20G  23% /
tmpfs                             tmpfs  2.0G     0  2.0G   0% /dev/shm
tmpfs                             tmpfs  5.0M     0  5.0M   0% /run/lock
/dev/sda2                         ext4   2.0G  125M  1.7G   7% /boot
tmpfs                             tmpfs  393M  4.0K  393M   1% /run/user/0
/dev/sdb1                         ext4   9.8G   24K  9.3G   1% /mnt
```
