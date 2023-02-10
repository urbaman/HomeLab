# Stop Ceph node

```bash
ceph osd set noscrub
ceph osd set nodeep-scrub

ceph status
```

## set cluster in maintenance mode with :

```bash
ceph osd set noout
```

## Set ceph back to working state

```bash
ceph osd unset noout
ceph osd unset noscrub
ceph osd unset nodeep-scrub

ceph status
```
