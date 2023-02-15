# Setup a Cluster

## Best practices

1. Define at least three subnets/vlans

 * Web gui subnet (the one with the gateway - vlan100, Management)
 * Cluster communication subnet (vlan20 - Proxmox Cluster) - Choosen during cluster setup
 * Migration network (vlan80 - Proxmox VM Migration network) - This is to separate the network through which Proxmox performs VM migration between nodes, set in Datacenter/Options

2. Optional (see Ceph): define another two subnets/vlans for Ceph

 * Ceph Storage (vlan30 - Ceph Storage - at least 10G)
 * Ceph Public (vlan40 - Ceph Public)
