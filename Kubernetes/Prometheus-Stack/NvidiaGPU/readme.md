# Install the 


```bash
helm repo add https://nvidia.github.io/dcgm-exporter/helm-charts
helm show values gpu-helm-charts/dcgm-exporter > dcgm-exporter-values.yaml
vi dcgm-exporter-values.yaml
```

Set the release: kube-prometheus-stack in the serviceMonitor additionalLabels:

```bash
serviceMonitor:
  enabled: true
  interval: 15s
  honorLabels: false
  additionalLabels:
    #monitoring: prometheus
    release: kube-prometheus-stack
```

Set a label to the GPU-provisioned nodes

```bash
kubectl label nodes <your-node-name> gpu=<gpu device> # "QuadroP2200" for me
```

Check the nodes' labels, with the Nvidia Device Plugin the GPU-provisioned nodes should have something like `"nvidia.com/gpu.product": "NVIDIA-DEVICE-NAME"`

Set the nodeSelector to `nvidia.com/gpu.product: "NVIDIA-DEVICE-NAME"`

```bash
nodeSelector:
  #node: gpu
  nvidia.com/gpu.product: "NVIDIA-DEVICE-NAME"
```

```bash
helm upgrade -i -n nvidia-device-plugin dcgm-exporter gpu-helm-charts/dcgm-exporter --values dcgm-exporter-values.yaml
```

Add the grafana dashboard

```bash
kubectl create configmap grafana-dashboard-nvidia-gpu --from-file=grafana-nvidia-gpu.json
kubectl label configmap grafana-dashboard-nvidia-gpu grafana_dashboard="1"
```
