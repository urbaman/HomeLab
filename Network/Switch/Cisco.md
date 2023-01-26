# Cisco Switch 3172TQ

## Cisco Switch first settings

Use only those that fit your needs.

### Connect to the device via console

Use a terminal emulation software such as PuTTY and connect to the console of the switch. You will get the initial command prompt “Switch>”

Type “enable” and hit enter. You will get into privileged EXEC mode (“Switch#”)

Now, get into Global Configuration Mode:

```bash
Switch# configure terminal
Switch(config)#
```

### Set up a hostname for the particular switch to distinguish it in the network

```bash
Switch(config)# hostname access-switch1
access-switch1(config)#
```

### Set the Switch domain

```bash
ip domain-name DOMAIN.COM
```

### Configure an administration password (enable secret password)

```bash
access-switch1(config)# enable secret somestrongpass
```

The password above will be used to enter into Privileged EXEC mode as described in Step 1 above.

### Configure a password for Telnet and Console access

It is a very good security practice to lock-down all access lines of a switch with a password. Although it is much better to configure an external AAA server (for centralized Authentication Authorization and Accounting), in this article we will just configure a password on each access line (VTY lines for Telnet and Console line):

```bash
access-switch1(config)# line vty 0 15
access-switch1(config-line)# password strongtelnetpass
access-switch1(config-line)# login
access-switch1(config-line)# exit
access-switch1(config)#
```

```bash
access-switch1(config)# line console 0
access-switch1(config-line)# password strongconsolepass
access-switch1(config-line)# login
access-switch1(config-line)# exit
access-switch1(config)#
```

### Define which IP addresses are allowed to access the switch via Telnet

```bash
access-switch1(config)# ip access-list standard TELNET-ACCESS
access-switch1(config-std-nacl)# permit 10.1.1.100
access-switch1(config-std-nacl)# permit 10.1.1.101
access-switch1(config-std-nacl)# exit
```

Apply the access list to Telnet VTY Lines

```bash
access-switch1(config)# line vty 0 15
access-switch1(config-line)# access-class TELNET-ACCESS in
access-switch1(config-line)# exit
access-switch1(config)#
```

### Set admin password for SSH access

```bash
change-password OLD_PASSWORD, NEW-PASSWORD, CONFIRM-PASSWORD
```

### Set up DNS look up

```bash
access-switch1# config t
access-switch1(config)# vrf context management
access-switch1(config)# ip domain-name mycompany.com
access-switch1(config)# ip name-server 172.68.0.10     
access-switch1(config)# ip domain-lookup
```

### Setup ntp for clock suncronization

```bash
feature ntp
ntp server 0.it.pool.ntp.org prefer use-vrf management
ntp server 1.it.pool.ntp.org use-vrf management
ntp server 2.it.pool.ntp.org use-vrf management
ntp server 3.it.pool.ntp.org use-vrf management
```

### Set the Timezone

```bash
clock timezone CET 1 0
```

### Set the day-light summertime settings if needed

```bash
clock summer-time CET 4 Sun Mar 02:00 4 Sun Oct 03:00
```

### Assign IP address to the switch for management

Management IP is assigned to mgmt by default

```bash
access-switch1(config)# interface mgmt0
access-switch1(config-if)# ip address 10.1.1.200 255.255.255.0
access-switch1(config-if)# exit
```

### Assign default gateway to the switch

```bash
access-switch1(config)# vrf context management
access-switch1(config-vrf)# ip route 0.0.0.0/0 10.0.1.1
access-switch1(config-vrf)# exit
```

### Disable unneeded ports on the switch

```bash
access-switch1(config)# interface 0/25-48
access-switch1(config-if-range)# shutdown
access-switch1(config-if-range)# exit
access-switch1(config)#
```

### Configure Layer2 VLANs and assign ports to the them

By default, all physical ports of the switch belong to the native VLAN1. One of the most important functions of an Ethernet switch is to segment the network into multiple Layer2 VLANs (with each VLAN belonging to a different Layer3 subnet).

In order to do the above Layer2 segmentation you need to create additional VLANs from the default VLAN1 and then assign physical ports to these new vlans. Let’s create two new vlans (VLAN2 and VLAN3) and assign two ports to each one.

#### First create the Layer2 VLANs on the switch

```bash
access-switch1(config)# vlan 2
access-switch1(config-vlan)# name TEACHERS
access-switch1(config-vlan)# exit
```

```bash
access-switch1(config)# vlan 3
access-switch1(config-vlan)# name STUDENTS
access-switch1(config-vlan)# exit
```

#### Now assign the physical ports to each VLAN. Ports 1-2 are assigned to VLAN2 and ports 3-4 to VLAN3

```bash
access-switch1(config)# interface 0/1-2
access-switch1(config-if-range)# switchport mode access
access-switch1(config-if-range)# switchport access vlan 2
access-switch1(config-if-range)# exit
```

```bash
access-switch1(config)# interface fa 0/3-4
access-switch1(config-if-range)# switchport mode access
access-switch1(config-if-range)# switchport access vlan 3
access-switch1(config-if-range)# exit
```

### Set MTU to 9000 (max for pfSense)

```bash
policy-map type network-qos jumbo
  class type network-qos class-default
          mtu 9000
system qos
  service-policy type network-qos jumbo
```

### Save the configuration

```bash
access-switch1(config)# exit
access-switch1# copy running-config startup-config
```

The above command to save the configuration can also be accomplished with  copy run start

The above are some steps that can be followed for basic set-up of a Cisco switch. Of course there are more things you can configure (such as SNMP servers, NTP, AAA, Vlan trunking protocol, 802.1q Trunk ports, Layer 3 inter-vlan routing etc) but those depend on the requirements of each particular network.

### Some Useful “Show” Commands

After configuring the basic steps above, let’s see some useful commands to monitor your configuration or troubleshoot possible problems:

```bash
access-switch1# show run  # Displays the current running configuration
access-switch1# show interfaces  # Displays the configuration of all interfaces and the status of each one
access-switch1# show vlan  # Displays all vlan numbers, names, ports associated with each vlan etc
access-switch1# show interface status  # Displays status of interfaces, speed, duplex etc
access-switch1# show mac address-table  # Displays current MAC address table and which MAC address is learned on each interface
```
