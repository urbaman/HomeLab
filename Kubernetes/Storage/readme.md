# Storage planning

I did try some different storage solutions for K8s running on Proxmox VMs, and probably I have found out some useful things to think about before deploying your cluster for choosing the best storage solution for your persistent volumes.

- If you have Ceph running on Proxmox, the best way is Ceph CSI or Rook with external Cluster based on Proxmox's. You can have both RBD (block devices) and CephSF (File access) possibilities.
- All other Storage solutions (GlusterFS, Longhorn, NFS, OpenEBS) are better on baremetal and not on virtual disks, you should only try with disk passthrough so that you avoid adding Ceph or ZFS lags to Gluster, NFS, Longhorn or OpenEBS lags. They need other resources to run (other VMs, k8s resources, ...) and only add to overall 

Proxmox Clusters with enough nodes and resources (proper 10G+ networks for Ceph, disks) can set up Ceph and/or CephFS pools and access them via Rook or Ceph CSI, and that's the better quite native solution, so I think you should go there if you can, both for stability, reliability and performances.

After many tries, I decided to stick with Ceph, as it's native with Proxmox and because of my disk distribution (mostly 2 disks of same tipe and dimension per node, so can't passthrough them to all of the k8s workers, and it could be a real pain to differentiate them by tipe/dimension on all of those other storage solutions).

This way I can easily get all that I need on a stable and performant distributed storage.
