# Proxmox Backup Monitoring with prometheus and pushgateway

## Install prometheus pushgateway on kubernetes

```bash
helm show values prometheus-community/prometheus-pushgateway > pushgateway-values.yaml
```

Define the namespace to monitoring, the Service Monitor to true and add the release=kube-prometheus-stack additional label to the Service Monitor.

```bash
helm upgrade -i prometheus-pushgateway prometheus-community/prometheus-pushgateway --values pushgateway-values.yaml -n monitoring
```

Then, create the ingressroute from the yaml template

## Proxmox Backup Server exporter

Install the Proxmox Backup Server Exporter from [here](https://github.com/rare-magma/pbs-exporter)

**Note:** you'll probably need to tweak it a little bit to make it properly source the conf file.
{: .note}

## Add grafana Dashboard

```bash
kubectl create configmap grafana-dashboard-proxmox-bs --from-file=grafana-proxmox-backup-server.json
kubectl label configmap grafana-dashboard-proxmox-bs grafana_dashboard="1"
```
