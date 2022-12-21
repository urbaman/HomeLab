# Interfaces check

ip link set dev eth1 up

for i in $( ls /sys/class/net ); do echo -n $i: ; cat /sys/class/net/$i/carrier; done

Edit the file /etc/udev/rules.d/70-persistent-net.rules to change the interface name of a network device.

The names of the network devices are listed in this file as follows:

# PCI device 0x11ab:0x4363 (sky2)
SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*",
ATTR{address}=="00:00:00:00:00:00",ATTR{dev_id}=="0x0", ATTR{type}=="1",
KERNEL=="eth*", NAME="eth0"
