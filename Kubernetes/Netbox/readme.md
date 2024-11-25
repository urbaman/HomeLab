# Netbox

## External behind Traefik

```bash
kubectl apply -f ig-netbox-external.yaml
```

## Internal via helm chart

Create a 40 hexadecimal characters string for the admin API token (not working):

```bash
head -c 40 /dev/urandom | xxd -p -c 40
```

Get the values:

```bash
helm show values --devel oci://ghcr.io/netbox-community/netbox-chart/netbox > netbox-values.yaml
vi netbox-values.yaml
```

Set the following values:

```yaml
superuser:
  password: <passoword>
#  apiToken: <APItoken (see above)>
email:
  server: mail.domain.com
  port: <25,465,587>
  username: <smtp username>
  password: <smtp password>
  useSSL: <true (465)>
  useTLS: <true/false>
  from: netbox@domain.com
loginRequired: true
plugins:
  - plugin1
  - plugin2
pluginsConfig:
  plugin1:
    custom: true
  plugin2:
    custom: true
preferIPv4: true
timeZone: Europe/Rome
dateFormat: 'j N, Y'
shortDateFormat: 'd-m-Y'
timeFormat: 'G:i'
shortTimeFormat: 'H:i:s'
dateTimeFormat: 'j N, Y G:i'
shortDateTimeFormat: 'd-m-Y H:i'
persistence:
  storageClass: asustornas1-nfs
reportsPersistence:
  enabled: true
  storageClass: asustornas1-nfs
metrics:
  enabled: true
  serviceMonitor:
    enabled: false
    additionalLabels:
      release: kube-prometheus-stack
postgresql:
  enabled: false
externalDatabase:
  host: postgresql.postgresql.svc.cluster.local
  port: 5432
  database: netbox
  username: netbox
  password: "<password>"
  existingSecretName: ""
  existingSecretKey: postgresql-password
redis:
  enabled: false
tasksRedis:
  database: "1"
  host: redis-master.redis.svc.cluster.local
  password: "<password>"
```

Install the chart and the treaefik ingressRoute:

```bash
helm upgrade -i netbox -n netbox --create-namespace --devel oci://ghcr.io/netbox-community/netbox-chart/netbox -f netbox-values.yaml
kubectl apply -f ig-netbox.yaml
```
