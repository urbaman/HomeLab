# Cloud-init template

Just create a vm from the iso you want. Install and setup whatever you want.

## Install qemu-guest-agent

```bash
sudo apt install qemu-guest-agent
sudo service qemu-guest-agent start
```

## Set user to passwordless sudo

```bash
sudo visudo
```

add at the end:

```bash
ubuntu (ALL)=ALL NOPASSWD:ALL
```

## Set your timezone

```bash
sudo timedatectl set-timezone Europe/Rome
```

## install and setup whatever else you want (docker, kubernetes, ...)

Do it.

## Write a script to run at boot to resize the disk

```bash
sudo vi /usr/sbin/disk-resize
```

```bash
#!/bin/bash

growpart /dev/sda 3
pvresize /dev/sda3
lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
resize2fs /dev/ubuntu-vg/ubuntu-lv
```

```bash
sudo chmod +x /usr/sbin/disk-resize
sudo crontab -e
```

add

```bash
@reboot /usr/sbin/disk-resize > /dev/null 2>&1
```

## Clear cloud-init

```bash
sudo cloud-init clean --logs
```
