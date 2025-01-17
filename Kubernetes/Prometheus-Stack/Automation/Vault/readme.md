# Vault Monitoring

Enable metrics and serviceMonitors.

```bash
kubectl create configmap grafana-dashboard-vault -n monitoring --from-file=grafana-vault.json
kubectl label configmap grafana-dashboard-vault -n monitoring grafana_dashboard="1"
```
