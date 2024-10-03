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
ubuntu ALL=(ALL) NOPASSWD:ALL
```

## Set your timezone

```bash
sudo timedatectl set-timezone Europe/Rome
```

## Install and setup whatever else you want (postfix, unattended-updates, docker, kubernetes, ...)

Do it.

## Create script to run at new template creation, to customize postfix and other mail settings to the new desired settings (mailto)

```bash
nano /usr/sbin/mailcustomize
```

```bash
#!/bin/bash

#Define the variables
HOST=$(hostname | tr '[:lower:]' '[:upper:]')

#Set alias mail for root
sed -i "s/ubuntu@domain.com/$1/g" /etc/aliases
newaliases

service postfix restart

#Set mailto for unattended-upgrades
sed -i "s/ubuntu@domain.com/$1/g" /etc/apt/apt.conf.d/50unattended-upgrades
service unattended-upgrades restart
```

```bash
chmod +x /usr/sbin/mailcustomize
```

The script runs with just one input variable, the mail to assign to the root as alias. All mails will be forwarded to that address.

## Set script to run at boot to customize FROM mails to HOST

```bash
sudo vi /usr/sbin/mailsetfrom
```

```bash
#!/bin/bash

#Define the variables
HOST=$(hostname | tr '[:lower:]' '[:upper:]')

#Set custom FROM for mails
sed -i "s/UBUNTU/$HOST/g" /etc/postfix/smtp_header_checks
postmap /etc/postfix/smtp_header_checks
service postfix restart
```

```bash
sudo chmod +x /usr/sbin/mailsetfrom
sudo crontab -e
```

add

```bash
@reboot /usr/sbin/mailsetfrom > /dev/null 2>&1
```


## Setup ssh keys to be used between clones

```bash
ssh-keygen
```

Copy the content of the public key to .ssh/authorized_keys

```bash
cat .ssh/id_rsa.pub >> .ssh/authorized_keys
```

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
