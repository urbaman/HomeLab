# Gitea Monitoring

Enable metrics and serviceMonitors.

```bash
kubectl create configmap grafana-dashboard-gitea -n monitoring --from-file=grafana-gitea.json
kubectl label configmap grafana-dashboard-gitea -n monitoring grafana_dashboard="1"
```
