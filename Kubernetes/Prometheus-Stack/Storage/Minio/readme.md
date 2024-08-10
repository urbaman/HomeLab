# Monitoring Minio

scrape_configs:
- job_name: minio-job
  bearer_token: <token-cluster>>
  metrics_path: /minio/v2/metrics/cluster
  scheme: https
  static_configs:
  - targets: [minio-s3.domain.com]

scrape_configs:
- job_name: minio-job-node
  bearer_token: <token-node>>
  metrics_path: /minio/v2/metrics/node
  scheme: https
  static_configs:
  - targets: [minio-s3.domain.com]

scrape_configs:
- job_name: minio-job-bucket
  bearer_token: <token-bucket>>
  metrics_path: /minio/v2/metrics/bucket
  scheme: https
  static_configs:
  - targets: [minio-s3.domain.com]

scrape_configs:
- job_name: minio-job-resource
  bearer_token: <token-resource>
  metrics_path: /minio/v2/metrics/resource
  scheme: https
  static_configs:
  - targets: [minio-s3.domain.com]

kubectl create secret generic minio-metrics-bearer-tokens -n minio \
    --from-literal=cluster-token=<token-cluster> \
    --from-literal=node-token=<token-node> \
    --from-literal=bucket-token=<token-bucket> \
    --from-literal=resource-token=<token-resource>
