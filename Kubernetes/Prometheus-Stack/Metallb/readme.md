# Monitoring metallb

After implementing the prometheus-stack, get the prometheus-operator.yaml file from the github project, add the label ```release: kube-prometheus-stack``` to both the Pod Monitors and apply (see the given yamls)

Install the grafana dashboard, download the json from grafana, create the configmap in the monitoring namespace and label it.

```bash
kubectl create configmap grafana-dashboard-metallb -n monitoring --from-file=grafana-mllb.json
kubectl label configmap grafana-dashboard-metallb -n monitoring grafana_dashboard="1"
```
