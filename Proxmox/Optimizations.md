# General OS Optimizations

Thanks to [Extremeshock](https://github.com/extremeshok/xshok-proxmox) among the others

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
systemctl enable powertop
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
nano /etc/apt/sources.list
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
apt purge ntp openntpd systemd-timesyncd
```

### Install fixes for APT

```bash
apt install apt-transport-https debian-archive-keyring ca-certificates curl
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

### Disable VIM VISUAL mode

```bash
nano /root/.vimrc
```

```bash
set mouse-=a
```

### Install and run lynis for security audit

```bash
apt install lynis
lynis audit system
```

### Install and configure zfs autosnapshot (12x5min, 24xh, 7xd, 4xw, 3xm)

```bash
apt install zfs-auto-snapshot
sed -i 's|--keep=[0-9]*|--keep=12|g' /etc/cron.d/zfs-auto-snapshot
sed -i 's|*/[0-9]*|*/5|g' /etc/cron.d/zfs-auto-snapshot
sed -i 's|--keep=[0-9]*|--keep=24|g' /etc/cron.hourly/zfs-auto-snapshot
sed -i 's|--keep=[0-9]*|--keep=7|g' /etc/cron.daily/zfs-auto-snapshot
sed -i 's|--keep=[0-9]*|--keep=4|g' /etc/cron.weekly/zfs-auto-snapshot
sed -i 's|--keep=[0-9]*|--keep=3|g' /etc/cron.monthly/zfs-auto-snapshot
```

### Install and configure ksmtuned

```bash
apt install ksmtuned
vi /etc/ksmtuned.conf
```

Set KSM_SLEEP_MSEC and KSM_THRES_COEF as follos:

- if RAM_SIZE_GB < 16 then: KSM_THRES_COEF=50, KSM_SLEEP_MSEC=80
- if RAM_SIZE_GB < 32 then: KSM_THRES_COEF=40, KSM_SLEEP_MSEC=60
- if RAM_SIZE_GB < 64 then: KSM_THRES_COEF=30, KSM_SLEEP_MSEC=40
- if RAM_SIZE_GB < 128 then: KSM_THRES_COEF=20, KSM_SLEEP_MSEC=20
- else: KSM_THRES_COEF=10, KSM_SLEEP_MSEC=10

### Install kernel headers

```bash
apt install pve-headers module-assistant
```

### Disable rpcbind

```bash
systemctl disable rpcbind
systemctl stop rpcbind
```

### Set pigz to replace gzip in vzdump, 2x faster gzip compression

```bash
sed -i "s/#pigz:.*/pigz: 1/" /etc/vzdump.conf
apt install pigz
cat  <<EOF > /bin/pigzwrapper
#!/bin/sh
PATH=/bin:\$PATH
GZIP="-1"
exec /usr/bin/pigz "\$@"
EOF
mv -f /bin/gzip /bin/gzip.original
cp -f /bin/pigzwrapper /bin/gzip
chmod +x /bin/pigzwrapper
chmod +x /bin/gzip
```

### Install and configure fail2ban to protect webui

```bash
apt install fail2ban
cat <<EOF > /etc/fail2ban/filter.d/proxmox.conf
[Definition]
failregex = pvedaemon\[.*authentication failure; rhost=<HOST> user=.* msg=.*
ignoreregex =
EOF
cat <<EOF > /etc/fail2ban/jail.d/proxmox.conf
[proxmox]
enabled = true
port = https,http,8006,8007
filter = proxmox
logpath = /var/log/daemon.log
maxretry = 3
# 1 hour
bantime = 3600
findtime = 600
EOF
```

### Optimizations for sysctl

```bash
nano /etc/modules
```

add the following lines

```bash
# Network
ip_conntrack
```

```bash
nano /etc/sysctl.conf
```

```bash
# Enable restart on kernel panic, kernel oops and hardlockup
kernel.core_pattern=/var/crash/core.%t.%p
# Reboot on kernel panic afetr 10s
kernel.panic=10
# Panic on kernel oops, kernel exploits generally create an oops
kernel.panic_on_oops=1
# Panic on a hardlockup
kernel.hardlockup_panic=1

# Increase max user watches
fs.inotify.max_user_watches=1048576
fs.inotify.max_user_instances=1048576
fs.inotify.max_queued_events=1048576

# Increase kernel max Key limit
kernel.keys.root_maxkeys=1000000
kernel.keys.maxkeys=1000000

# Memory Optimising
## Bugfix: reserve 1024MB memory for system
vm.min_free_kbytes=1048576
vm.nr_hugepages=72
# (Redis/MongoDB)
vm.max_map_count=262144
vm.overcommit_memory = 1

# TCP BBR congestion control
net.core.default_qdisc=fq
net.ipv4.tcp_congestion_control=bbr

# TCP fastopen
net.ipv4.tcp_fastopen=3

# Network optimizing
net.core.netdev_max_backlog=8192
net.core.optmem_max=8192
net.core.rmem_max=16777216
net.core.somaxconn=8151
net.core.wmem_max=16777216
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.all.log_martians = 0
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.all.secure_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.conf.default.log_martians = 0
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.default.secure_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.icmp_ignore_bogus_error_responses = 1
net.ipv4.ip_local_port_range=1024 65535
net.ipv4.tcp_base_mss = 1024
net.ipv4.tcp_challenge_ack_limit = 999999999
net.ipv4.tcp_fin_timeout=10
net.ipv4.tcp_keepalive_intvl=30
net.ipv4.tcp_keepalive_probes=3
net.ipv4.tcp_keepalive_time=240
net.ipv4.tcp_limit_output_bytes=65536
net.ipv4.tcp_max_syn_backlog=8192
net.ipv4.tcp_max_tw_buckets = 1440000
net.ipv4.tcp_mtu_probing = 1
net.ipv4.tcp_rfc1337=1
net.ipv4.tcp_rmem=8192 87380 16777216
net.ipv4.tcp_sack=1
net.ipv4.tcp_slow_start_after_idle=0
net.ipv4.tcp_syn_retries=3
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_tw_reuse = 0
net.ipv4.tcp_wmem=8192 65536 16777216
net.netfilter.nf_conntrack_generic_timeout = 60
net.netfilter.nf_conntrack_helper=0
net.netfilter.nf_conntrack_max = 524288
net.netfilter.nf_conntrack_tcp_timeout_established = 28800
net.unix.max_dgram_qlen = 4096

# Bugfix: high swap usage with low memory usage
vm.swappiness=10

# Max FS Optimising
fs.nr_open=12000000
fs.file-max=9000000
fs.aio-max-nr=524288
```

```bash
sysctl -p
```

```bash
cat <<EOF >> /etc/security/limits.d/99-limits.conf
# Increase max FD limit / ulimit
* soft     nproc          1048576
* hard     nproc          1048576
* soft     nofile         1048576
* hard     nofile         1048576
root soft     nproc          unlimited
root hard     nproc          unlimited
root soft     nofile         unlimited
root hard     nofile         unlimited
EOF
```

### Set systemd ulimits

```bash
echo "DefaultLimitNOFILE=256000" >> /etc/systemd/system.conf
echo "DefaultLimitNOFILE=256000" >> /etc/systemd/user.conf
echo 'session required pam_limits.so' >> /etc/pam.d/common-session
echo 'session required pam_limits.so' >> /etc/pam.d/runuser-l
```

### Set ulimit for the shell user

```bash
echo "ulimit -n 256000" >> /root/.profile
```

### Optimize Logrotate

```bash
cat <<EOF > /etc/logrotate.conf
daily
su root adm
rotate 7
create
compress
size=10M
delaycompress
copytruncate

include /etc/logrotate.d
EOF
```

```bash
systemctl restart logrotate
```

### Optimize journald

```bash
cat <<EOF > /etc/systemd/journald.conf
[Journal]
# Store on disk
Storage=persistent
# Don't split Journald logs by user
SplitMode=none
# Disable rate limits
RateLimitInterval=0
RateLimitIntervalSec=0
RateLimitBurst=0
# Disable Journald forwarding to syslog
ForwardToSyslog=no
# Journald forwarding to wall /var/log/kern.log
ForwardToWall=yes
# Disable signing of the logs, save cpu resources.
Seal=no
Compress=yes
# Fix the log size
SystemMaxUse=64M
RuntimeMaxUse=60M
# Optimise the logging and speed up tasks
MaxLevelStore=warning
MaxLevelSyslog=warning
MaxLevelKMsg=warning
MaxLevelConsole=notice
MaxLevelWall=crit
EOF
```

```bash
systemctl restart systemd-journald.service
journalctl --vacuum-size=64M --vacuum-time=1d;
journalctl --rotate
```

### Make sure entropy is managed

```bash
apt install haveged
```

```bash
cat <<EOF > /etc/default/haveged
#   -w sets low entropy watermark (in bits)
DAEMON_ARGS="-w 1024"
EOF
```

```bash
systemctl daemon-reload
systemctl enable haveged
```

### Increase vzdump backup speed

```bash
sed -i "s/#bwlimit:.*/bwlimit: 0/" /etc/vzdump.conf
sed -i "s/#ionice:.*/ionice: 5/" /etc/vzdump.conf
```

### Customise bashrc (thanks broeckca)

```bash
cat <<EOF >> /root/.bashrc
export HISTTIMEFORMAT="%d/%m/%y %T "
export PS1='\u@\h:\W \$ '
alias l='ls -CF'
alias la='ls -A'
alias ll='ls -alF'
alias ls='ls --color=auto'
source /etc/profile.d/bash_completion.sh
export PS1="\[\e[31m\][\[\e[m\]\[\e[38;5;172m\]\u\[\e[m\]@\[\e[38;5;153m\]\h\[\e[m\] \[\e[38;5;214m\]\W\[\e[m\]\[\e[31m\]]\[\e[m\]\\$ "
EOF
```

```bash
echo "source /root/.bashrc" >> /root/.bash_profile
```

### Optimise ZFS arc size accoring to memory size

- if RAM_SIZE_GB < 16 then: MY_ZFS_ARC_MIN=536870911, MY_ZFS_ARC_MAX=536870912
- if RAM_SIZE_GB < 32 then: MY_ZFS_ARC_MIN=1073741823, MY_ZFS_ARC_MAX=1073741824
- else: MY_ZFS_ARC_MIN=$((RAM_SIZE_GB * 1073741824 / 16)), MY_ZFS_ARC_MAX=$((RAM_SIZE_GB * 1073741824 / 8))

- if MY_ZFS_ARC_MIN < 536870911 then: MY_ZFS_ARC_MIN=536870911
- if MY_ZFS_ARC_MAX < 536870912 then: MY_ZFS_ARC_MAX=536870912

```bash
cat <<EOF > /etc/modprobe.d/99-zfsarc.conf
# Use 1/8 RAM for MAX cache, 1/16 RAM for MIN cache, or 1GB
options zfs zfs_arc_min=$MY_ZFS_ARC_MIN
options zfs zfs_arc_max=$MY_ZFS_ARC_MAX

# use the prefetch method
options zfs l2arc_noprefetch=0

# max write speed to l2arc
# tradeoff between write/read and durability of ssd (?)
# default : 8 * 1024 * 1024
# setting here : 500 * 1024 * 1024
options zfs l2arc_write_max=524288000
options zfs zfs_txg_timeout=60
EOF
```

### Propagate the settings

```bash
update-initramfs -u -k all
pve-efiboot-tool refresh
apt autoremove && apt autoclean
```
