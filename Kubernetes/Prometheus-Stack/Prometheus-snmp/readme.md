# Prometheus snmp exporter for Arista Switch and Dell IDRAC (valid for any other snmp monitored device)

## Generate your snmp.yml

Use the promethueus snmp exporter generator to produce your custom snmp.yml file
See the MIB files for Dell IDRAC and Arista Switch, eventually use yours.

## Create two configmaps from the snmp.yml file

Set the username and passwords (both passwords) for both the modules in `snmp.yml` file

```bash
kubectl create configmap prometheus-snmp-exporter-idrac -n monitoring --from-file snmp.yml
kubectl create configmap prometheus-snmp-exporter-arista-switch -n monitoring --from-file snmp.yml
```

## Deploy the given snmp exporters (customizing the yaml files to your needs)

Set the IPs to the devices in both files

```bash
helm upgrade -i prometheus-snmp-exporter-idrac prometheus-community/prometheus-snmp-exporter -f snmp-exporter-idrac.yaml -n monitoring
helm upgrade -i prometheus-snmp-exporter-arista-switch prometheus-community/prometheus-snmp-exporter -f snmp-exporter-arista-switch.yaml -n monitoring
```

## Add the dashboards to Grafana

```bash
kubectl create configmap grafana-dashboard-arista-switch -n monitoring --from-file=grafana-arista.json
kubectl label configmap grafana-dashboard-arista-switch -n monitoring grafana_dashboard="1"
kubectl create configmap grafana-dashboard-dell-idrac -n monitoring --from-file=grafana-idrac.json
kubectl label configmap grafana-dashboard-dell-idrac -n monitoring grafana_dashboard="1"
```
