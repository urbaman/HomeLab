# ArgoCD Monitoring

Enable metrics and serviceMonitors.

```bash
kubectl create configmap grafana-dashboard-argocd -n monitoring --from-file=grafana-argocd.json
kubectl label configmap grafana-dashboard-argocd -n monitoring grafana_dashboard="1"
kubectl create configmap grafana-dashboard-argocd-application-overview -n monitoring --from-file=grafana-argocd-application-overview.json
kubectl label configmap grafana-dashboard-argocd-application-overview -n monitoring grafana_dashboard="1"
kubectl create configmap grafana-dashboard-argocd-notifications-overview -n monitoring --from-file=grafana-argocd-notifications-overview.json
kubectl label configmap grafana-dashboard-argocd-notifications-overview -n monitoring grafana_dashboard="1"
kubectl create configmap grafana-dashboard-argocd-operational-overview -n monitoring --from-file=grafana-argocd-operational-overview.json
kubectl label configmap grafana-dashboard-argocd-operational-overview -n monitoring grafana_dashboard="1"
```
