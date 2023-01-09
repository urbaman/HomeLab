# General OS Optimizations

## Powertop

```bash
apt install powertop
nano /etc/systemd/system/powertop.service
```

```bash
[Unit]
Description=Powertop tunings

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/sbin/powertop --auto-tune

[Install]
WantedBy=multi-user.target
```

```bash
systemctl enable powetop
systemctl start powertop
```

## CPU governor

```bash
apt install linux-cpupower
nano /etc/systemd/system/cpupower.service
```

```bash
[Unit]
Description=CPU powersave

[Service]
Type=oneshot
ExecStart=/usr/bin/cpupower -c all frequency-set -g ondemand

[Install]
WantedBy=multi-user.target
```

```bash
systemctl enable cpupower
systemctl start cpupower
```

## Set variables for subsequent optimizations

```bash
OS_CODENAME="$(grep "VERSION_CODENAME=" /etc/os-release | cut -d"=" -f 2 | xargs )"
RAM_SIZE_GB=$(( $(vmstat -s | grep -i "total memory" | xargs | cut -d" " -f 1) / 1024 / 1000))
```

### Force APT to use IPv4

```bash
echo -e "Acquire::ForceIPv4 \"true\";\\n" > /etc/apt/apt.conf.d/99-xs-force-ipv4
```

### Add debian non-free repositories

```bash
nano /etc/apt/spources.list
```

Add ```non-free``` to the debian repositories

```bash
deb http://ftp.it.debian.org/debian bullseye main contrib non-free
deb http://ftp.it.debian.org/debian bullseye-updates main contrib non-free
# security updates
deb http://security.debian.org bullseye-security main contrib non-free
```

```bash
apt update && apt dist-upgrade
```

### Purge unwanted packages

```bash
purge ntp openntpd systemd-timesyncd
```

### Install fixes for APT

```bash
install apt-transport-https debian-archive-keyring ca-certificates curl
apt update && apt dist-upgrade
```

### Install missing packages

```bash
apt install zfsutils-linux proxmox-backup-restore-image chrony
```

### Install common system utilities

```bash
apt install axel build-essential curl dialog dnsutils dos2unix git gnupg-agent grc htop iftop iotop iperf ipset iptraf mlocate msr-tools nano net-tools software-properties-common sshpass tmux unzip vim vim-nox wget whois zip
```

### Install and run lynis for security audit

```bash
apt install lynis
lynis audit system
```bash

### Install and configure zfs autosnapshot (12x5min, 24xh, 7xd, 4xw, 3xm)

```bash
apt install zfs-auto-snapshot
sed -i 's|--keep=[0-9]*|--keep=12|g' /etc/cron.d/zfs-auto-snapshot
sed -i 's|*/[0-9]*|*/5|g' /etc/cron.d/zfs-auto-snapshot
sed -i 's|--keep=[0-9]*|--keep=24|g' /etc/cron.hourly/zfs-auto-snapshot
sed -i 's|--keep=[0-9]*|--keep=7|g' /etc/cron.daily/zfs-auto-snapshot
sed -i 's|--keep=[0-9]*|--keep=4|g' /etc/cron.weekly/zfs-auto-snapshot
sed -i 's|--keep=[0-9]*|--keep=3|g' /etc/cron.monthly/zfs-auto-snapshot
```bash

### Install and configure ksmtuned

```bash
apt install ksmtuned
vi /etc/ksmtuned.conf
```bash

Set KSM_SLEEP_MSEC and KSM_THRES_COEF as follos:

- if RAM_SIZE_GB < 16 then
        # start at 50% full
        KSM_THRES_COEF=50
        KSM_SLEEP_MSEC=80
- if RAM_SIZE_GB < 32 then
        # start at 60% full
        KSM_THRES_COEF=40
        KSM_SLEEP_MSEC=60
- if RAM_SIZE_GB < 64 then
        # start at 70% full
        KSM_THRES_COEF=30
        KSM_SLEEP_MSEC=40
- if RAM_SIZE_GB < 128 then
        # start at 80% full
        KSM_THRES_COEF=20
        KSM_SLEEP_MSEC=20
- else
        # start at 90% full
        KSM_THRES_COEF=10
        KSM_SLEEP_MSEC=10
