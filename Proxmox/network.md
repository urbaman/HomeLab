# Interfaces check

ip link set dev eth1 up

for i in $( ls /sys/class/net ); do echo -n $i: ; cat /sys/class/net/$i/carrier; done
