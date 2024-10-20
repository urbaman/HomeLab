# Vault Monitoring

Enable metrics and serviceMonitors.

```bash
kubectl create configmap grafana-dashboard-vault --from-file=grafana-vault.json
kubectl label configmap grafana-dashboard-vault grafana_dashboard="1"
```
