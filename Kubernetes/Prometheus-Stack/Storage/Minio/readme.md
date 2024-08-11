# Monitoring Minio

Install the mc tool if you didn't already:

```bash
sudo curl https://dl.min.io/client/mc/release/linux-amd64/mc --create-dirs -o /usr/bin/mc
sudo chmod +x /usr/bin/mc
mc --help
```

Press ESC to exit the help, then set an alias and check the API:

```bash
mc alias set ALIAS_NAME HOSTNAME ACCESS_KEY SECRET_KEY
mc admin info myminio
```

Use it to generate scrape configs for the different endpoints:

```bash
mc admin prometheus generate ALIAS [node,bucket,resource]
```

You'll get something like this (depending on the endpoint scrape config generated)

```yaml
scrape_configs:
- job_name: minio-job
  bearer_token: <token-cluster>>
  metrics_path: /minio/v2/metrics/cluster
  scheme: https
  static_configs:
  - targets: [minio-s3.domain.com]
```

```yaml
scrape_configs:
- job_name: minio-job-node
  bearer_token: <token-node>>
  metrics_path: /minio/v2/metrics/node
  scheme: https
  static_configs:
  - targets: [minio-s3.domain.com]
```

```yaml
scrape_configs:
- job_name: minio-job-bucket
  bearer_token: <token-bucket>>
  metrics_path: /minio/v2/metrics/bucket
  scheme: https
  static_configs:
  - targets: [minio-s3.domain.com]
```

```yaml
scrape_configs:
- job_name: minio-job-resource
  bearer_token: <token-resource>
  metrics_path: /minio/v2/metrics/resource
  scheme: https
  static_configs:
  - targets: [minio-s3.domain.com]
```

Create a secret with the different tokens:

```bash
kubectl create secret generic minio-metrics-bearer-tokens -n minio \
    --from-literal=cluster-token=<token-cluster> \
    --from-literal=node-token=<token-node> \
    --from-literal=bucket-token=<token-bucket> \
    --from-literal=resource-token=<token-resource>
```

Then create the servicemonitors and the dashboards:

```bash
kubectl apply -f sm-minio.yaml
kubectl create configmap grafana-dashboard-minio -n monitoring --from-file=minio-dashboard.json
kubectl label configmap grafana-dashboard-minio -n monitoring grafana_dashboard="1"
kubectl create configmap grafana-dashboard-minio-bucket -n monitoring --from-file=minio-bucket-dashboard.json
kubectl label configmap grafana-dashboard-minio-bucket -n monitoring grafana_dashboard="1"
kubectl create configmap grafana-dashboard-minio-node -n monitoring --from-file=minio-node-dashboard.json
kubectl label configmap grafana-dashboard-minio-node -n monitoring grafana_dashboard="1"
kubectl create configmap grafana-dashboard-minio-replication -n monitoring --from-file=minio-replication-dashboard.json
kubectl label configmap grafana-dashboard-minio-replication -n monitoring grafana_dashboard="1"
```
