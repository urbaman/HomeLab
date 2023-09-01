# Gluster monitoring

## Install the exporter

```bash
sudo curl -fsSL https://github.com/kadalu/gluster-metrics-exporter/releases/latest/download/install.sh | sudo bash -x
sudo systemctl enable gluster-metrics-exporter
sudo systemctl start gluster-metrics-exporter
```

## Install the endpoits, service and servicemonitor

In the yaml, we use relabeling to make the hostname value readable and manageable. Adjust to your situation.

```bash
kubectl apply -f sm-gluster-exporter.yaml
```

## Install the Grafana dashboard
