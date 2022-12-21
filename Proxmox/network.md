# Interfaces check

To check which interface is actually linked to which physical port, enable all of the interfaces. For each one:

```bash
ip link set dev eth1 up
```

Then, check where the following script gives a "1" result with the cable connected, moving the cable through the physical ports and taking note of the port-interface link.

```bash
for i in $( ls /sys/class/net ); do echo -n $i: ; cat /sys/class/net/$i/carrier; done
```

Go to ```/etc/systemd/network``` and create some "n-interface-rename.link" files (where 'n' is < 99 and 'interface' is the interface old name for reference), one for each interface to rename, mapping the MAC address with an interface name. Be aware to use completely new names to avoid overlap.

```bash
[Match]
MACAddress=00:a0:de:63:7a:e6

[Link]
Name=wan0
```

Or:

```bash
[Match]
MACAddress=00:a0:de:63:7a:e6

[Link]
Name=lan10
```

Then, rename the interfaces in ```/etc/network/interfaces``` and reload with ```ifreload -a```
