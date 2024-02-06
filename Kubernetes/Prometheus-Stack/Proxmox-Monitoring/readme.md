# Proxmox, Prometheus, Grafana

## On proxmox nodes

From the Datacenter/Permissions view on the proxmox webgui, create a user (prometheus@pve) with Audit on "\" (all resources)

From the shell:

```bash
apt install python3 python3-pip
pip3 install prometheus-pve-exporter
vi /etc/prometheus/pve.yml
```

```bash
default:
    user: user@pve
    password: your_password_here
    verify_ssl: false
```

```bash
vi /etc/systemd/system/prometheus-pve-exporter.service
```

```bash
[Unit]
Description=Prometheus exporter for Proxmox VE
Documentation=https://github.com/znerol/prometheus-pve-exporter

[Service]
Restart=always
ExecStart=/usr/local/bin/pve_exporter /etc/prometheus/pve.yml

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload
systemctl start prometheus-pve-exporter
systemctl enable prometheus-pve-exporter
```

Check going to http:<PVE_NODE_IP/HOST>:9221/pve?target=<IP_OF_A_PVE_NODE>

## On  Kubernetes cluster

Define endpoints, service and service monitor with the template yaml.

Then, add the dashboard to grafana and create the configmap.

```bash
kubectl create configmap grafana-dashboard-proxmox --from-file=grafana-proxmox.json -n monitoring
kubectl label configmap grafana-dashboard-proxmox grafana_dashboard="1" -n monitoring
```
