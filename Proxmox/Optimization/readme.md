# Optimization scripts

## powertop

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

## CPU governor
