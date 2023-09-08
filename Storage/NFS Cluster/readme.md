# NFS cluster with corosync and pacemaker

## Requirements

1. Glusterfs cluster (on the same machines): [Gluster](https://github.com/urbaman/HomeLab/tree/main/Storage/Glusterfs)

## Install and setup the cluster

### On all Nodes, Install Pacemaker and configure some settings

```bash
sudo apt -y install pacemaker pcs resource-agents
sudo systemctl stop pacemaker corosync
sudo systemctl enable --now pcsd
```

#### Set cluster admin password

```bash
sudo passwd hacluster
Changing password for user hacluster.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```

### On a Node, Configure basic Cluster settings

#### Authorize among nodes

```bash
sudo pcs host auth node01.srv.world node02.srv.world
Username: hacluster
Password:
node01.srv.world: Authorized
node02.srv.world: Authorized
```

#### Configure the cluster

```bash
sudo pcs cluster setup ha_cluster node01.srv.world node02.srv.world
No addresses specified for host 'node01.srv.world', using 'node01.srv.world'
No addresses specified for host 'node02.srv.world', using 'node02.srv.world'
Destroying cluster on hosts: 'node01.srv.world', 'node02.srv.world'...
node02.srv.world: Successfully destroyed cluster
node01.srv.world: Successfully destroyed cluster
Requesting remove 'pcsd settings' from 'node01.srv.world', 'node02.srv.world'
node01.srv.world: successful removal of the file 'pcsd settings'
node02.srv.world: successful removal of the file 'pcsd settings'
Sending 'corosync authkey', 'pacemaker authkey' to 'node01.srv.world', 'node02.srv.world'
node01.srv.world: successful distribution of the file 'corosync authkey'
node01.srv.world: successful distribution of the file 'pacemaker authkey'
node02.srv.world: successful distribution of the file 'corosync authkey'
node02.srv.world: successful distribution of the file 'pacemaker authkey'
Sending 'corosync.conf' to 'node01.srv.world', 'node02.srv.world'
node01.srv.world: successful distribution of the file 'corosync.conf'
node02.srv.world: successful distribution of the file 'corosync.conf'
Cluster has been successfully set up.
```

#### Start services for the cluster

```bash
sudo pcs cluster start --all
node01.srv.world: Starting Cluster...
node02.srv.world: Starting Cluster...
```

#### Set auto-start

```bash
sudo pcs cluster enable --all
node01.srv.world: Cluster Enabled
node02.srv.world: Cluster Enabled
```

#### Show status

```bash
sudo pcs cluster status
Cluster Status:
 Cluster Summary:
   * Stack: corosync
   * Current DC: node01.srv.world (version 2.1.2-ada5c3b36e2) - partition with quorum
   * Last updated: Thu Sep 15 00:17:19 2022
   * Last change:  Thu Sep 15 00:17:13 2022 by hacluster via crmd on node01.srv.world
   * 2 nodes configured
   * 0 resource instances configured
 Node List:
   * Online: [ node01.srv.world node02.srv.world ]

PCSD Status:
  node01.srv.world: Online
  node02.srv.world: Online

sudo pcs status corosync

Membership information
----------------------
    Nodeid      Votes Name
         1          1 node01.srv.world (local)
         2          1 node02.srv.world
```

### If you'd like to remove all cluster settings to initalize, run like follows

```bash
sudo pcs cluster stop --all
node01.srv.world: Stopping Cluster (pacemaker)...
node02.srv.world: Stopping Cluster (pacemaker)...
node01.srv.world: Stopping Cluster (corosync)...
node02.srv.world: Stopping Cluster (corosync)...

sudo pcs cluster destroy --all
Warning: Unable to load CIB to get guest and remote nodes from it, those nodes will not be deconfigured.
node02.srv.world: Stopping Cluster (pacemaker)...
node01.srv.world: Stopping Cluster (pacemaker)...
node02.srv.world: Successfully destroyed cluster
node01.srv.world: Successfully destroyed cluster
```

## Setup the fencing device using iSCSI (working? -> No)

### On all Cluster Nodes, Install SCSI Fence Agent

```bash
sudo apt -y install fence-agents-base
```

### Configure Fencing on a Node

[sda] of the example below is the storage from iSCSI target.

#### Confirm disk ID

```bash
sudo ll /dev/disk/by-id | grep sda | grep wwn
lrwxrwxrwx 1 root root   9 Sep 15 00:30 wwn-0x60014054a2b171ec9974ef5a736a642d -> ../../sda
```

#### Set fencing

[scsi-shooter] : any name
[pcmk_host_list=***] : specify cluster nodes
[devices=***] : disk ID

```bash
sudo pcs stonith create scsi-shooter fence_scsi pcmk_host_list="node01.srv.world node02.srv.world" devices=/dev/disk/by-id/wwn-0x60014054a2b171ec9974ef5a736a642d meta provides=unfencing
```

#### Show config

```bash
sudo pcs stonith config scsi-shooter
 Resource: scsi-shooter (class=stonith type=fence_scsi)
  Attributes: devices=/dev/disk/by-id/wwn-0x60014054a2b171ec9974ef5a736a642d pcmk_host_list="node01.srv.world node02.srv.world"
  Meta Attrs: provides=unfencing
  Operations: monitor interval=60s (scsi-shooter-monitor-interval-60s)
```

#### Show status (OK if the status of fence device is [Started])

```bash
sudo pcs status
Cluster name: ha_cluster
Cluster Summary:
  * Stack: corosync
  * Current DC: node01.srv.world (version 2.1.2-ada5c3b36e2) - partition with quorum
  * Last updated: Thu Sep 15 00:42:03 2022
  * Last change:  Thu Sep 15 00:41:27 2022 by root via cibadmin on node01.srv.world
  * 2 nodes configured
  * 1 resource instance configured

Node List:
  * Online: [ node01.srv.world node02.srv.world ]

Full List of Resources:
  * scsi-shooter        (stonith:fence_scsi):    Started node01.srv.world

Daemon Status:
  corosync: active/enabled
  pacemaker: active/enabled
  pcsd: active/enabled
```

### Try to test fencing

From a different node:

```bash
sudo pcs status
Cluster name: ha_cluster
Cluster Summary:
  * Stack: corosync
  * Current DC: node01.srv.world (version 2.1.2-ada5c3b36e2) - partition with quorum
  * Last updated: Thu Sep 15 00:42:55 2022
  * Last change:  Thu Sep 15 00:41:27 2022 by root via cibadmin on node01.srv.world
  * 2 nodes configured
  * 1 resource instance configured

Node List:
  * Online: [ node01.srv.world node02.srv.world ]

Full List of Resources:
  * scsi-shooter        (stonith:fence_scsi):    Started node01.srv.world

Daemon Status:
  corosync: active/enabled
  pacemaker: active/enabled
  pcsd: active/enabled
```

#### Fencing

```bash
sudo pcs stonith fence node01.srv.world
Node: node01.srv.world fenced
```

#### target node turns to [OFFLINE] and it will be restarted

```bash
sudo pcs status
Cluster name: ha_cluster
Cluster Summary:
  * Stack: corosync
  * Current DC: node02.srv.world (version 2.1.2-ada5c3b36e2) - partition with quorum
  * Last updated: Thu Sep 15 00:43:48 2022
  * Last change:  Thu Sep 15 00:41:27 2022 by root via cibadmin on node01.srv.world
  * 2 nodes configured
  * 1 resource instance configured

Node List:
  * Online: [ node02.srv.world ]
  * OFFLINE: [ node01.srv.world ]

Full List of Resources:
  * scsi-shooter        (stonith:fence_scsi):    Started node02.srv.world

Daemon Status:
  corosync: active/enabled
  pacemaker: active/enabled
  pcsd: active/enabled
```

#### After rebooting, if you manually start the node, do like follows

```bash
sudo pcs cluster start node01.srv.world
```

## Fencing through diskless SBD and software watchdog

### Install softdog module

```bash
sudo echo "softdog" >> /etc/modules
```

Check if softdog is blacklisted

```bash
grep blacklist /lib/modprobe.d/* /etc/modprobe.d/* |grep softdog
```

If there's any result, comment or delete the `blacklist softdog` line(s), then enable and check softdog

```bash
sudo modprobe softdog
sudo lsmod | grep softdog
softdog                16384  0
ls -l /dev/watchdog*

crw-rw---- 1 postgres postgres  10, 130 Sep 11 12:53 /dev/watchdog
crw------- 1 root     root     245,   0 Sep 11 12:53 /dev/watchdog0
```

### Install and setup SBD

Prepare the cluster properties

```bash
sudo pcs property set stonith-watchdog-timeout=35
```

Install the sbd package

```bash
sudo apt install sbd
```

Open the file `/etc/default/sbd` and use the following entries:

```bash
SBD_PACEMAKER=yes
SBD_STARTMODE=always
SBD_DELAY_START=no
SBD_WATCHDOG_DEV=/dev/watchdog
SBD_WATCHDOG_TIMEOUT=35
```

> **Important**: SBD_WATCHDOG_TIMEOUT for diskless SBD and QDevice
>If you use QDevice with diskless SBD, the SBD_WATCHDOG_TIMEOUT value must be greater than QDevice's sync_timeout value, or SBD will time out and fail to start.
>
>The default value for sync_timeout is 30 seconds. Therefore, set SBD_WATCHDOG_TIMEOUT to a greater value, such as 35.

```bash
systemctl enable sbd
sudo pcs cluster stop --all
sudo pcs cluster start --all
sudo pcs property | grep have-watchdog
 have-watchdog: true
```

## Setup Cluster resources (VIP, NFS server)

### On all Cluster Nodes, Install NFS tools

sudo apt -y install nfs-kernel-server nfs-common

### Prepare the NFS filesystem

On the (one of the) shared volume(s), create the NFS directories needed by the cluster:

```bash
sudo mkdir /HDD5T/nfsshare
sudo mkdir /HDD5T/nfsshare/exports
sudo mkdir /HDD5T/nfsshare/exports/HDD5T
```

And add eventually one for every other gluster volume:

```bash
sudo mkdir /HDD5T/nfsshare/exports/SDD2T
```

In this case, we also need to mount of the other volumes in the relative mount position.

```bash
sudo vi /etc/systemd/system/HDD5T-nfsshare-exports-SDD2T.mount
```

```bash
[Unit]
Description=Mount glusterfs HDD5T/nfsshare/exports/SDD2T

[Mount]
What=thisserverfullhostname:/volumename
Where=/mountpath
Type=glusterfs
Options=_netdev,auto

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable HDD5T-nfsshare-exports-SDD2T.mount
```

Finally, define the NFS filesystem permissions.

```bash
sudo chown -R nobody:nogroup /HDD5T/nfsshare/exports
sudo chmod -R 775 /HDD5T/nfsshare/exports
```

### Add the NFS server resource

```bash
sudo pcs resource create nfs_server ocf:heartbeat:nfsserver nfs_shared_infodir=/HDD5T/nfsshare/nfsinfo nfs_no_notify=true --group nfs_group
```

### Add the VIP resource

```bash
sudo pcs resource create nfs_vip ocf:heartbeat:IPaddr2 ip=VIP cidr_netmask=24 --group nfs_group
```

### Add the NFS exports (the root and the real exports)

Use the same values in clientspec, directory, options and fsid that you would use in the normal NFS `/etc/exports` file.
To expose the same export to different ips/cidrs, simply add a resource for every occurrence, setting the clientspec accordingly.

```bash
sudo pcs resource create nfs_export_root ocf:heartbeat:exportfs clientspec=10.0.50.0/24 options=rw,sync,root_squash,all_squash,no_subtree_check directory=/HDD5T/nfsshare/exports fsid=0 --group nfs_group
sudo pcs resource create nfs_export_hdd5t ocf:heartbeat:exportfs clientspec=10.0.50.0/24 options=rw,sync,root_squash,all_squash,no_subtree_check directory=/HDD5T/nfsshare/exports/HDD5T fsid=1 --group nfs_group
sudo pcs resource create nfs_export_sdd2t ocf:heartbeat:exportfs clientspec=10.0.50.0/24 options=rw,sync,root_squash,all_squash,no_subtree_check directory=/HDD5T/nfsshare/exports/SDD2T fsid=2 --group nfs_group
```

### Update resource parameters

```bash
sudo pcs resource update nfs_export_root ocf:heartbeat:exportfs clientspec=10.0.50.0/24 options=rw,sync,no_root_squash,no_all_squash,no_subtree_check directory=/HDD5T/nfsshare/exports fsid=0 --group nfs_group
sudo pcs resource update nfs_export_hdd5t ocf:heartbeat:exportfs clientspec=10.0.50.0/24 options=rw,sync,no_root_squash,no_all_squash,no_subtree_check directory=/HDD5T/nfsshare/exports/HDD5T fsid=1 --group nfs_group
sudo pcs resource update nfs_export_sdd2t ocf:heartbeat:exportfs clientspec=10.0.50.0/24 options=rw,sync,no_root_squash,no_all_squash,no_subtree_check directory=/HDD5T/nfsshare/exports/SDD2T fsid=2 --group nfs_group
```

## Set static ports to NFS secondary services

Edit /etc/default/nfe.conf and add specific ports (maybe from 3265 to 3268) to lockd (same ports on port and udp port, 32768), mountd (32767) and statd (port 32766, outgoing-port 32765).

Save and restart the service. Now you can explicitly open the ports on the firewall (both tcp and udp)
