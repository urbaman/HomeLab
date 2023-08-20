# NFS cluster with corosync and pacemaker

Follow [this link](https://www.server-world.info/en/note?os=Ubuntu_22.04&p=pacemaker&f=1)

- After enabling the cluster, change the corosync.conf file setting the nodes' IPs instead of the hostnames in the nodelist.
- When creating the fencing device through iscsi, put the devices setting between "".
