# Uptime Kuma for service uptime monitoring

## Install Uptime Kuma through helm

Once Helm has been set up correctly, add the repo as follows:

```bash
helm repo add uptime-kuma https://dirsigler.github.io/uptime-kuma-helm
helm repo update
helm show values uptime-kuma/uptime-kuma > uptime-kuma-values.yaml
```

If you had already added this repo earlier, run helm repo update to retrieve the latest versions of the packages. You can then run helm search repo uptime-kuma to see the charts.

Change the values to install the latest image, on desired storageClass.

```bash
helm upgrade -i uptime-kuma uptime-kuma/uptime-kuma -n monitoring -f uptime-kuma-values.yaml
```

## Setting up serviceMonitor for monitoring

After the first deployment, login to uptime-kuma and create an API key, the go back to the values file and set the serviceMonitor enabled, the basicauth user to whatever and the password to the API key, then upgrade the helm deployment.

## Expose through IngressRoute

Create and install the ssl cert and ingressroute

```bash
kubectl apply -f ig-uptimekuma.yaml
```
