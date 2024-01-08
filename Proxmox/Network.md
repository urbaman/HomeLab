# Network for Proxmox

## Interface check

To check which interface is actually linked to which physical port, enable all of the interfaces. For each one:

```bash
ip link set dev eth1 up
```

Then, check where the following script gives a "1" result with the cable connected, moving the cable through the physical ports and taking note of the port-interface link.

```bash
for i in $( ls /sys/class/net ); do echo -n $i: ; cat /sys/class/net/$i/carrier; done
```

## Set the network (bonds, bridges, vlans)

See the example /etc/network/interfaces file [here](https://github.com/urbaman/HomeLab/tree/main/Proxmox/network/interfaces)