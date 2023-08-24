# Ceph management

## Stop Ceph node

```bash
ceph osd set noscrub
ceph osd set nodeep-scrub

ceph status
```

### set cluster in maintenance mode with

```bash
ceph osd set noout
```

### Set ceph back to working state

```bash
ceph osd unset noout
ceph osd unset noscrub
ceph osd unset nodeep-scrub

ceph status
```

## Ceph rules, maps and crushmaps

### Custom OSD Classes

Just write the new class (nvme, sas...) in the right place in the popup while creating the OSD.

### Custom Crushmap

```bash
ceph osd crush rule create-replicated <cruchmap_name>  default  host <class_name>
```

Then, create the pool with the crushmap

Enjoy!

### Renaming a pool

```bash
ceph osd pool rename OLD NEW
```

### Renaming a CephFS

```bash
ceph fs rename Cephfs_SAS900MB Cephfs_SAS900GB --yes-i-really-mean-it
```

### Renaming OSDs class

```bash
ceph osd crush class rename OLD NEW
```

### Renaming a crush rule

Create the new class, change the class to the new one on the pools, delete the old class

```bash
ceph osd crush rule rm OLD
```
