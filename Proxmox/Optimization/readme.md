# Optimization scripts

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
