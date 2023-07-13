# Fritzbox exporter for Prometheus Monitoring

## Preparation

1. Add a user to the Fritzbox with admin access.
2. Activate UPnP in Home Network settings
3. Make sure the k8s nodes can ping the host fritz.box (just the domanin without any subdomain/host) to the Fritbox IP (add it to your internal DNS server or to the hosts file of the nodes)

## Apply the prom-fritzbox.yaml

Customize the environment variables in the prom-fritzbox.yaml matching your environment (use the user previously added to te Fritzbox)

```bash
kubectl apply -f prom-fritzbox.yaml
```

## Create the Grafana dashboard

```bash
kubectl create configmap grafana-dashboard-fritzbox --from-file=grafana-fritzbox.json
kubectl label configmap grafana-dashboard-fritzbox grafana_dashboard="1"
```
