# Truenas SCALE

## Machines

10.0.50.21 truenas1.urbaman.it

10.0.50.22 truenas2.urbaman.it

10.0.50.23 truenas3.urbaman.it

10.0.50.24 truecommand.urbaman.it

## AD - Nethserver

10.0.50.25 ad.urbaman.it

## Gluster

10.0.70.21 gluster1.urbaman.it

10.0.70.22 gluster2.urbaman.it

10.0.70.23 gluster3.urbaman.it

## Install TruenasSCALE (x3)

Remember to set a simple password to start

## Post-install

### User

Go to Credentials/Local Users and set password and mail for root

### System Settings/General

#### Localization

Change what needs to be changed (language, keyboard, timezone, date format...)

#### GUI

Set HTTP->HTTPS redirect, and Show Console Messages if you want.

### System Settings/General

#### SSH

Modify SSH settings to enable root login, set it to Run Automatically and Start it.

### Network

#### Interfaces

You need at least two interfaces for clustering, best on two different VLANs.

Go to Network, set interfaces one by one. Leave DHCP on the first one (for truenas) if you need, you'll need a fixed IP on the second one.

#### Global Configuration

Set hostname, domain. Additional domain should be your AD domain if different.

Set nameservers, set the first to the Domain Controller, the second to the real DNS server if different.

Set the default gateway.

Set the hostnames db to the IPs/hostnames of the TruenasSCALE ecosystem (TruenasSCALEs, truecommand, AD - Nethserver, Gluster interfaces)

### Pools

Go to Storage and define your Pools.

GO to System Setitngs, Advances, and move the System Dataset to the boot-pool
