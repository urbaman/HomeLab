# Monitoring the NFS cluster

## Install the node exporter

```bash
sudo apt-get update
sudo apt-get install prometheus-node-exporter -y
sudo systemctl status prometheus-node-exporter
```

## Install the endpoits, service and servicemonitor

```bash
kubectl apply -f sm-nodeexporter.yaml
```

## Install the grafana dashboard

```bash
kubectl create configmap grafana-dashboard-nfs -n monitoring --from-file=grafana-nfs.json
kubectl label configmap grafana-dashboard-nfs -n monitoring grafana_dashboard="1"
```
