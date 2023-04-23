# Arista switches

## Enter the configuration status

```bash
localhost>enable
localhost#config
```

## Saving the configuration

```bash
localhost(config)#copy running-config startup-config
```

## Add admin password

```bash
localhost(config)#username admin secret pxq123
```

## Default route and management IP

```bash
localhost(config)#ip route 0.0.0.0/0 192.0.2.1
localhost(config)#interface management 1
localhost(config-if-Ma1)#ip address 192.0.2.8/24
localhost(config)#exit
```

## Envoronment management

### Show environment

```bash
localhost>show environment [all, cooling [detail], power [detail], temperature [module name] [detail]]
```

### Fan speed management

```bash
localhost(config)#environment fan-speed override [percent 30-100, auto]
```

## Switch hostname, FQDN and name-server

```bash
localhost(config)#hostname [hostname]
hostname(config)#ip domain-name [domain.com]
hostname(config)#ip name-server [xx.xx.xx.xx]
```

## Clock management

```bash
hostname(config)#show clock
hostname(config)#closck timezone [timezone, eg Europe/Rome]
hostname(config)#clock set [date, eg 08:15:24 14 Jan 2013]
hostname(config)#ntp server [server] [prefer]
```

## Third party transceivers

```bash
hostname(config)#bash touch /mnt/flash/enable3px
```

You'll probably need to restart the switch

## Ports management

### VLANS

```bash
hostname(config)#vlan XX
hostname(config-vlan-XX)#name [name]
hostname(config-vlan-XX)#exit
```

### Port channels (LACP, Bond)

```bash
hostname(config)#interface ethernet [x; x-y; x,y; x-y,z]
hostname(config-if-EtX)#channel-group X mode active
hostname(config-if-EtX)#exit
```

### Set interface to VLAN, access

```bash
hostname(config)#interface [ethernet, port-channel] [x; x-y; x,y; x-y,z]
hostname(config-if-[EtX, PoX])#switchport mode access
hostname(config-if-[EtX, PoX])#switchport access vlan XX
```

## Set interface to VLAN(s), trunk

```bash
hostname(config)#interface [ethernet, port-channel] [x; x-y; x,y; x-y,z]
hostname(config-if-[EtX, PoX])#switchport mode trunk
hostname(config-if-[EtX, PoX])#switchport trunk allowed vlan [x; x-y; x,y; x-y,z]
hostname(config-if-[EtX, PoX])#switchport trunk allowed vlan add [x; x-y; x,y; x-y,z]
hostname(config-if-[EtX, PoX])#switchport trunk native vlan X
```

