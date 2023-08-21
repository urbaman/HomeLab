# NFS cluster with corosync and pacemaker

[Kubernetes](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/04-Kubernetes)

```bash

```

Follow [this link](https://www.server-world.info/en/note?os=Ubuntu_22.04&p=pacemaker&f=1)
... and [this](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_high_availability_clusters/assembly_configuring-active-passive-nfs-server-in-a-cluster-configuring-and-managing-high-availability-clusters)

## Requirements

1. Glusterfs cluster (on the same machines): [Gluster](https://github.com/urbaman/HomeLab/tree/main/Storage/Glusterfs)
2. iSCSI Target (different machine), iSCSI initiators (the same machines): [iSCSI Target](https://github.com/urbaman/HomeLab/tree/main/Storage/iSCSI)

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

## Setup the fencing device using iSCSI (working?)

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

## Setup Cluster resources (VIP, NFS server)

### On all Cluster Nodes, Install NFS tools

sudo apt -y install nfs-kernel-server nfs-common

### On a Node add NFS resource

#### Current status

sudo pcs status
Cluster name: ha_cluster
Cluster Summary:
  * Stack: corosync
  * Current DC: node01.srv.world (version 2.1.2-ada5c3b36e2) - partition with quorum
  * Last updated: Thu Sep 15 01:52:09 2022
  * Last change:  Thu Sep 15 01:52:02 2022 by root via cibadmin on node01.srv.world
  * 2 nodes configured
  * 2 resource instances configured

Node List:
  * Online: [ node01.srv.world node02.srv.world ]

Full List of Resources:
  * scsi-shooter        (stonith:fence_scsi):    Started node01.srv.world
  * Resource Group: ha_group:
    * lvm_ha    (ocf:heartbeat:LVM-activate):    Started node01.srv.world

Daemon Status:
  corosync: active/enabled
  pacemaker: active/enabled
  pcsd: active/enabled

#### Create a directory for NFS filesystem

sudo mkdir /home/nfs-share

#### Set nfsserver resource

[nfs_daemon] : any name
[nfs_shared_infodir=***] : specify a directory that NFS server related files are located

sudo pcs resource create nfs_daemon ocf:heartbeat:nfsserver nfs_shared_infodir=/home/nfs-share/nfsinfo nfs_no_notify=true --group ha_group

#### Set IPaddr2 resource (VIP for client access)

sudo pcs resource create nfs_vip ocf:heartbeat:IPaddr2 ip=10.0.0.60 cidr_netmask=24 --group ha_group
sudo pcs status
Cluster name: ha_cluster
Cluster Summary:
  * Stack: corosync
  * Current DC: node02.srv.world (version 2.1.2-ada5c3b36e2) - partition with quorum
  * Last updated: Thu Sep 15 04:13:57 2022
  * Last change:  Thu Sep 15 04:13:45 2022 by root via cibadmin on node01.srv.world
  * 2 nodes configured
  * 5 resource instances configured

Node List:
  * Online: [ node01.srv.world node02.srv.world ]

Full List of Resources:
  * scsi-shooter        (stonith:fence_scsi):    Started node01.srv.world
  * Resource Group: ha_group:
    * lvm_ha    (ocf:heartbeat:LVM-activate):    Started node01.srv.world
    * nfs_share (ocf:heartbeat:Filesystem):      Started node01.srv.world
    * nfs_daemon        (ocf:heartbeat:nfsserver):       Started node01.srv.world
    * nfs_vip   (ocf:heartbeat:IPaddr2):         Started node01.srv.world

Daemon Status:
  corosync: active/enabled
  pacemaker: active/enabled
  pcsd: active/enabled

### On an active Node NFS filesystem mounted, set exportfs setting.
#### create a directory for exportfs

sudo mkdir -p /home/nfs-share/nfs-root/share01

#### set exportfs resource

[nfs_root] : any name
[clientspec=*** options=*** directory=***] : exports setting
[fsid=0] : root point on NFSv4

sudo pcs resource create nfs_root ocf:heartbeat:exportfs clientspec=10.0.0.0/255.255.255.0 options=rw,sync,no_root_squash directory=/home/nfs-share/nfs-root fsid=0 --group ha_group

#### set exportfs resource

sudo pcs resource create nfs_share01 ocf:heartbeat:exportfs clientspec=10.0.0.0/255.255.255.0 options=rw,sync,no_root_squash directory=/home/nfs-share/nfs-root/share01 fsid=1 --group ha_group
sudo pcs status
Cluster name: ha_cluster
Cluster Summary:
  * Stack: corosync
  * Current DC: node02.srv.world (version 2.1.2-ada5c3b36e2) - partition with quorum
  * Last updated: Thu Sep 15 04:15:02 2022
  * Last change:  Thu Sep 15 04:14:55 2022 by root via cibadmin on node02.srv.world
  * 2 nodes configured
  * 7 resource instances configured

Node List:
  * Online: [ node01.srv.world node02.srv.world ]

Full List of Resources:
  * scsi-shooter        (stonith:fence_scsi):    Started node01.srv.world
  * Resource Group: ha_group:
    * lvm_ha    (ocf:heartbeat:LVM-activate):    Started node01.srv.world
    * nfs_share (ocf:heartbeat:Filesystem):      Started node01.srv.world
    * nfs_daemon        (ocf:heartbeat:nfsserver):       Started node01.srv.world
    * nfs_vip   (ocf:heartbeat:IPaddr2):         Started node01.srv.world
    * nfs_root  (ocf:heartbeat:exportfs):        Started node01.srv.world
    * nfs_share01       (ocf:heartbeat:exportfs):        Started node01.srv.world

Daemon Status:
  corosync: active/enabled
  pacemaker: active/enabled
  pcsd: active/enabled

sudo showmount -e
Export list for node01.srv.world:
/home/nfs-share/nfs-root         10.0.0.0/255.255.255.0
/home/nfs-share/nfs-root/share01 10.0.0.0/255.255.255.0

### Verify settings to access to virtual IP address with NFS from any client computer.

root@client:~# mount -t nfs4 10.0.0.60:share01 /mnt
root@client:~# df -hT /mnt
Filesystem         Type  Size  Used Avail Use% Mounted on
10.0.0.60:/share01 nfs4  9.8G  512K  9.3G   1% /mnt