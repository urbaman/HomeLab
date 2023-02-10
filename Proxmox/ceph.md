# Stop Ceph node

ceph osd set noscrub
ceph osd set nodeep-scrub

ceph status

    # set cluster in maintenance mode with :

ceph osd set noout
