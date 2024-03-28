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

### Increase maximum recovery & backfilling speed

Keep in mind that more recovery/backfilling resouces will add to the cluster resource usage, so use it only if and when needed

#### Set new parameters

Using these commands, you can adjust the parameters accordingly. The parameter values can vary depending on the system, e.g. a system with NVMes can have more OSD max backfills than a system with HDDs. These commands change the parameters on all available OSDs in the cluster.

```bash
ceph tell 'osd.*' injectargs '--osd-max-backfills 16'
ceph tell 'osd.*' injectargs '--osd-recovery-max-active 4'
```

#### Reset parameters

If necessary, you may want to set the default parameters again after the successful recovery. For this purpose, you should use the parameters from the section "Finding out the current parameters".

```bash
ceph tell 'osd.*' injectargs '--osd-max-backfills 1'
ceph tell 'osd.*' injectargs '--osd-recovery-max-active 0'
```
