# Haproxy monitoring with prometheus and grafana

## Haproxy prometheus exporter

Be sure to enable the native prometheus exporter in Haproxy (see [Haproxy Loadbalance](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/03-High-Availability))

## Prometheus

Apply Endpoint, Servce and ServiceMonitor

```bash
kubectl apply -f prom-haproxy1.yaml -f prom-haproxy2.yaml -f prom-haproxy3.yaml
```

Then, add Grafana dashboard

```bash
kubectl create configmap grafana-dashboard-haproxy -n monitoring --from-file=grafana-haproxy.json
kubectl label configmap grafana-dashboard-haproxy -n monitoring grafana_dashboard="1"
```

## Haproxy stats through traefik

```bash
kubectl apply -f traefik-haproxy1-svc.yaml -f traefik-haproxy1.yaml -f traefik-haproxy2-svc.yaml -f traefik-haproxy2.yaml -f traefik-haproxy3-svc.yaml -f traefik-haproxy3.yaml
```
