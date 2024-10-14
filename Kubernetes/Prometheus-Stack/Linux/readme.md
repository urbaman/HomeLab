# Node Exporter on linux

## On the node to monitor

```bash
curl -s https://api.github.com/repos/prometheus/node_exporter/releases/latest| grep browser_download_url|grep linux-amd64|cut -d '"' -f 4|wget -qi -
tar -xvf node_exporter*.tar.gz
cd  node_exporter*/
sudo cp node_exporter /usr/local/bin
cd
node_exporter --version
rm -rf node*
```

```bash
sudo tee /etc/systemd/system/node_exporter.service <<EOF
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=<USER>
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=default.target
EOF
```

```bash
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
```

The service is online on port 9100.

## Kubernetes ServiceMonitor and Grafana Dashboards

```bash
kubectl apply -f sm-node-exporter.yaml
kubectl create configmap grafana-dashboard-node-exporter -n monitoring --from-file=grafana-node-exporter.json
kubectl label configmap grafana-dashboard-node-exporter -n monitoring grafana_dashboard="1"
```
