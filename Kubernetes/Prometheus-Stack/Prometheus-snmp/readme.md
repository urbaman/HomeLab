# Prometheus snmp exporter for Arista Switch and Dell IDRAC (valid for any other snmp monitored device)

## Generate your snmp.yml

Use the promethueus snmp exporter generator to produce your custom snmp.yml file

## Create two configmaps from the snmp.yml file

```bash
kubectl create configmap prometheus-snmp-exporter-idrac -n monitoring --from-file snmp.yml
kubectl create configmap prometheus-snmp-exporter-arista-switch -n monitoring --from-file snmp.yml
```

## Deploy the given snmp exporters (customizing the yaml files to your needs)

```bash
helm install prometheus-snmp-exporter-idrac prometheus-community/prometheus-snmp-exporter -f snmp-exporter-idrac.yaml -n monitoring
helm install prometheus-snmp-exporter-arista-switch prometheus-community/prometheus-snmp-exporter -f snmp-exporter-arista-switch.yaml -n monitoring
```

## Add the dashboards to Grafana

```bash
kubectl create configmap grafana-dashboard-arista-switch --from-file=grafana-arista.json
kubectl label configmap grafana-dashboard-arista-switch grafana_dashboard="1"
kubectl create configmap grafana-dashboard-dell-idrac --from-file=grafana-idrac.json
kubectl label configmap grafana-dashboard-dell-idrac grafana_dashboard="1"
```bash
