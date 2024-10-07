# Install and use Intel GPUs

Prerequisites:

- Cert manager
- Node Feature Discovery

**Note:** do not use the microk8s kubectl command, as it's not compliant with the version management of the git needed by the following apply commands.

Install NFD rules:

```bash
kubectl apply -k 'https://github.com/intel/intel-device-plugins-for-kubernetes/deployments/nfd/overlays/node-feature-rules?ref=v0.31.0'
```

Deploy the plugin operator and the GPU custom resource (sample below):

```bash
kubectl apply -k 'https://github.com/intel/intel-device-plugins-for-kubernetes/deployments/operator/default?ref=v0.31.0'
kubectl apply -f https://raw.githubusercontent.com/intel/intel-device-plugins-for-kubernetes/main/deployments/operator/samples/deviceplugin_v1_gpudeviceplugin.yaml
```

```yaml
apiVersion: deviceplugin.intel.com/v1
kind: GpuDevicePlugin
metadata:
  name: gpudeviceplugin-sample
spec:
  image: intel/intel-gpu-plugin:0.31.0
  sharedDevNum: 10
  logLevel: 4
  enableMonitoring: true
  nodeSelector:
    intel.feature.node.kubernetes.io/gpu: "true"
```
