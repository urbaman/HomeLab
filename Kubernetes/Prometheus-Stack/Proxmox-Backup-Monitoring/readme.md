# Proxmox Backup Monitoring with prometheus and pushgateway

## Requisites

Install Pushgateway

## Exporter

Install the Proxmox Backup Server Exporter from [here](https://github.com/rare-magma/pbs-exporter)

**Note:** you'll probably need to tweak it a little bit to make it properly source the conf file.
{: .note}

## Add grafana Dashboard

```bash
kubectl create configmap grafana-dashboard-proxmox-bs -n monitoring --from-file=grafana-proxmox-backup-server.json
kubectl label configmap grafana-dashboard-proxmox-bs -n monitoring grafana_dashboard="1"
```

# PBS with PBS Exporter

Create an API Token in PBS, define the variables in the exporter-pbs.yaml

```bash
kubectl apply -f exporter-pbs.yaml
```

See [here](https://github.com/natrontech/pbs-exporter)

Add the dashboard

```bash
kubectl create configmap grafana-dashboard-pbs -n monitoring --from-file=grafana-pbs-v2.json
kubectl label configmap grafana-dashboard-pbs -n monitoring grafana_dashboard="1"
```
