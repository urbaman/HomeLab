# Virtual Machines management

## Change VM ID on Ceph

- Stop the VM
- Check the storage on Ceph

```bash
rbd -p <CephPool> ls | grep vm-OLDID
```

- Move all the devices (disks and cloudinit devices)

```bash
rbd -p <CephPool> mv vm-OLDID-xxx vm-NEWID-xxx
```

- Move the vm config file, and change all internal reference to old id

```bash
mv /etc/pve/nodes/NODE/qemu-server/OLDID.conf /etc/pve/nodes/NODE/qemu-server/NEWID.conf
sed -i "s/vm-OLDID/vm-NEWID/g" /etc/pve/nodes/NODE/qemu-server/NEWID.conf
```

- Restart the VM
