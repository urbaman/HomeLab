# ArgoCD Monitoring

Enable metrics and serviceMonitors.

```bash
kubectl create configmap grafana-dashboard-argocd --from-file=grafana-argocd.json
kubectl label configmap grafana-dashboard-argocd grafana_dashboard="1"
kubectl create configmap grafana-dashboard-argocd-application-overview --from-file=grafana-argocd-application-overview.json
kubectl label configmap grafana-dashboard-argocd-application-overview grafana_dashboard="1"
kubectl create configmap grafana-dashboard-argocd-notifications-overview --from-file=grafana-argocd-notifications-overview.json
kubectl label configmap grafana-dashboard-argocd-notifications-overview grafana_dashboard="1"
kubectl create configmap grafana-dashboard-argocd-operational-overview --from-file=grafana-argocd-operational-overview.json
kubectl label configmap grafana-dashboard-argocd-operational-overview grafana_dashboard="1"
```
