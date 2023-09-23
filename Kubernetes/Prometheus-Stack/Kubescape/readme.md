# Monitoring Kubescape

Remember to install kubescape with the serviceMonitor enabled: [Kubescape for observability and security scan](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Kubescape)

Edit the serviceMonitor adding the `release: kube-prometheus-stack` label, then add the `kubescape` namespace to the `kube-prometheus-stack-operator` deploymet and rollout restart it.

Finally, add the grafana dashboard

```bash
kubectl create configmap grafana-dashboard-kubescape --from-file=grafana-kubescape.json
kubectl label configmap grafana-dashboard-kubescape grafana_dashboard="1"
```
