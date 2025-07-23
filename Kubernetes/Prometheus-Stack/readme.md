# Monitoring with Prometheus-stack (Prometheus and Grafana) - Without Rancher

Start checking the bind address of kube-proxy, kube-scheduler, kube-control-monitor

```bash
kubectl edit cm kube-proxy -n kube-system
## Change from
    metricsBindAddress: 127.0.0.1:10249 ### <--- Too secure
## Change to
    metricsBindAddress: 0.0.0.0:10249
```

And reload the kube-proxy daemonset

```bash
kubectl rollout restart daemonset -n kube-system kube-proxy
```

On every control-panel, change bind address to 0.0.0.0:

```bash
sudo vi /etc/kubernetes/manifests/kube-scheduler.yaml
sudo vi /etc/kubernetes/manifests/kube-controller-manager.yaml
```

Also, change the etcd metrics listen address from 127.0.0.1 to 0.0.0.0:

```bash
sudo vi /etc/kubernetes/manifests/etcd.yaml
```

Then check the controller manager and scheduler pods, they should restart as soon as the system recognize the changed manifests, if not manually delete each scheduler and each controller manager pods in the kube-system namespace to make them load the new settings.

```bash
kubectl delete pod <pod-name> -n kube-system
```

**Note:** When using microk8s, the above settings about bind addresses are in `/var/snap/microk8s/current/args/`, specific files for each service. Remember the services are not pods, you'll need to set appropriate endpoints to properly scrape their metrics (see note later on).

Get the helm value file and add all your namespaces. Add others and upgrade the deploy if needed.

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm show values prometheus-community/kube-prometheus-stack > kube-prometheus-stack-values.yaml
```

Use the following command to get the namespaces with appended "      - ".

```bash
kubectl get ns  --no-headers -o custom-columns="'- ':metadata.name" | sed 's/^/      - /'
```

And add to the chart

```yaml
  namespaces:
    releaseNamespace: true
    additional:
      - calico-apiserver
      - calico-system
      - cert-manager
      - default
      - kube-node-lease
      - kube-public
      - kube-system
      - kubernetes-dashboard
      - kubescape
      - metallb-system
      - monitoring
      - portainer
      - rook-ceph
      - rook-ceph-external
      - teleport-cluster
      - tigera-operator
      - traefik
      - traefik-external
```

You can always change (add, delete, ...) the scraped namespaces editing the operator deployment

First, get the namespaces in comma delimitated format, then edit the --namespaces spec, replacing the comma separated string of namespaces with the new one, and restart the deployment

```bash
kubectl get ns -o jsonpath='{.items[*].metadata.name}' | sed -e "s/ /,/g"
kubectl edit -n monitoring deployment kube-prometheus-stack-operator
kubectl rollout restart -n monitoring deployment kube-prometheus-stack-operator
```

Disable the internal etcd and/or the internal kubeProxy scraping:

```yaml
kubeEtcd:
  enabled: false


kubeProxy:
  enabled: false

defaultRules:
  create: true
  rules:
    kubeProxy: false
    etcd: false # Only if using other cluster storage systems
```

And define Alertmanager notifications by email:

```yaml
    route:
      group_by: ['namespace']
      group_wait: 30s
      group_interval: 5m
      repeat_interval: 12h
      receiver: 'null'
      routes:
      - receiver: 'null'
        matchers:
          - alertname =~ "InfoInhibitor|Watchdog"
      - receiver: 'mail'
#        matchers:
#          - alertname != "InfoInhibitor|Watchdog"
    receivers:
    - name: 'null'
    - name: 'mail'
      email_configs:
      - to: 'tomail@domain.com'
        from: 'frommail@domain.com'
        smarthost: mail.domain.com:587
        auth_username: 'tomail@domain.com'
        auth_identity: 'tomail@domain.com'
        auth_password: 'password'
        send_resolved: true
        tls_config:
          insecure_skip_verify: true
        headers:
          subject: 'Prometheus Mail Alerts'
```

Under the grafana header, add some plugins:

```yaml
  plugins:
    - grafana-piechart-panel
    - grafana-clock-panel
    - vonage-status-panel
```

They will then be managed through the grafana config map:

```bash
kubectl edit cm -n monitoring kube-prometheus-stack-grafana
```

Set the `defaultDashboardsTimezone: browser` to get the right timezone in the dashboards.

Finally, set the storage specs for dynamic persistent storage:

```yaml
alertmanager:
  alertmanagerSpec:
    storage:
     volumeClaimTemplate:
       spec:
         storageClassName: rook-ceph-nvme2tb
         accessModes: ["ReadWriteOnce"]
         resources:
           requests:
             storage: 50Gi
prometheus:
  prometheusSpec:    
    storageSpec: 
     volumeClaimTemplate:
       spec:
         storageClassName: rook-ceph-nvme2tb
         accessModes: ["ReadWriteOnce"]
         resources:
           requests:
             storage: 50Gi
grafana:
  persistence:
    enabled: true
    type: pvc
    storageClassName: "asustornas1-nfs"
    accessModes:
      - ReadWriteOnce
    size: 20Gi
    finalizers:
      - kubernetes.io/pvc-protection
```

Finally, decide for a Grafana password

```yaml
grafana:
 adminPassword: prom-operator
```

Set the serviceMonitor, podMonitor, rules and probe to be scraped without needing specific lables (setting the, to true will need the `release: kube-prometheus-stack` label):

```yaml
prometheus:
  prometheusSpec:
    podMonitorSelectorNilUsesHelmValues: false
    ruleSelectorNilUsesHelmValues: false
    serviceMonitorSelectorNilUsesHelmValues: false
    probeSelectorNilUsesHelmValues: false
```

**Note:** When using microk8s, add the endpoints to proxy, scheduler and controller-manager, example:

```yaml
kubeScheduler:
  enabled: true

  ## If your kube scheduler is not deployed as a pod, specify IPs it can be found on
  ##
  endpoints:
   - IP1
   - IP2
   - IP3
```

And install:

```bash
helm upgrade -i --namespace monitoring --create-namespace kube-prometheus-stack prometheus-community/kube-prometheus-stack --values kube-prometheus-stack-values.yaml
```

## The alertmanager PVC problem

Probably the alertmanager pod will stuck in ContainerCreating state, waiting for a PV/PVC to be provisioned.

In that case, the PVC got messed up with a Selector {} spec that stops Longhorn from the volume provisioning: delete it, then rollout restart the statefullset

```bash
kubectl delete pvc -n monitoring alertmanager-kube-prometheus-stack-alertmanager-db-alertmanager-kube-prometheus-stack-alertmanager-0
kubectl rollout restart -n monitoring statefulset.apps/alertmanager-kube-prometheus-stack-alertmanager
```

## Monitoring other services/apps on other namespaces

Take a look at the helm values of the prometheus-stack implementation:

```bash
helm get values -n monitoring kube-prometheus-stack -a -o yaml > prom.yaml
```

Near the end you'll find something like:

```yaml
    serviceMonitorSelector: {}
    serviceMonitorSelectorNilUsesHelmValues: true
```

And the same for the podMonitorSelector. It means that the default `release: kube-prometheus-stack` is the label to add to the Service Monitors you'll need to create for the services to be monitored.

Putting `serviceMonitorSelectorNilUsesHelmValues` and/or `podMonitorSelectorNilUsesHelmValues` to false opens up to monitor all of the serviceMonitors and podMonitors in the monitored namesopaces. Do it if you need to scrape custom serviceMonitors and podMonitors.

### Example: Traefik (see more below for Traefik implementation)

Remember to set the metrics port expose setting to true when installing Traefik, then create the following Service Monitor:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: traefik-sm
  labels:
    traefik: http
    release: kube-prometheus-stack
spec:
  jobLabel: traefik
  selector:
    matchExpressions:
    - {key: app.kubernetes.io/name, operator: Exists}
  namespaceSelector:
    matchNames:
    - traefik
  endpoints:
  - port: metrics
    interval: 15s
```

To add a grafana dashboard, first install it in grafana, then download the json from grafana itself, and then create a configmap from it in the monitoring namespace

```bash
kubectl create configmap grafana-dashboard-traefik --from-file=/path_to/traefik_dashboard.json
kubectl label configmap grafana-dashboard-traefik grafana_dashboard="1"
```

## Exposing services with Traefik and cert-manager

Deploy the `ig-kube-prometheus-stack.yaml` file

```bash
kubectl apply -f ig-kube-prometheus-stack.yaml
```
