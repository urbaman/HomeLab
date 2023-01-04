# Optimization scripts

## Powertop

apt install powertop

nano /etc/systemd/system/powertop.service

[Unit]
Description=Powertop tunings

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/sbin/powertop --auto-tune

[Install]
WantedBy=multi-user.target

systemctl enable powetop
systemctl start powertop

## CPU governor

apt install linux-cpupower

nano /etc/systemd/system/cpupower.service

[Unit]
Description=CPU powersave

[Service]
Type=oneshot
ExecStart=/usr/bin/cpupower -c all frequency-set -g ondemand

[Install]
WantedBy=multi-user.target

systemctl enable cpupower
systemctl start cpupower
