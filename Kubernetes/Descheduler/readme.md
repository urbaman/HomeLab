# Running Descheduler

Descheduler helps keeping the cluster clean and balanced between nodes.

Look through the descheduler-values.yaml file for the plugin settings, [following the user guide](https://github.com/kubernetes-sigs/descheduler?tab=readme-ov-file#user-guide), then deploy with helm.

```bash
helm repo add descheduler https://kubernetes-sigs.github.io/descheduler/
helm repo update
helm upgrade -i descheduler -n kube-system descheduler/descheduler -f descheduler-values.yaml
```
