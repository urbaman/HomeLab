# Uptime Kuma for service uptime monitoring

## Install Uptime Kuma through helm

Once Helm has been set up correctly, add the repo as follows:

```bash
helm repo add uptime-kuma https://dirsigler.github.io/uptime-kuma-helm
helm repo update
```

If you had already added this repo earlier, run helm repo update to retrieve the latest versions of the packages. You can then run helm search repo uptime-kuma to see the charts.

To install the uptime-kuma chart, with the lates image:

```bash
helm upgrade my-uptime-kuma uptime-kuma/uptime-kuma --install --namespace monitoring --set image.tag=latest
```

## Expose through IngressRoute

Create and install the ssl cert and ingressroute

```bash
kubectl apply -f ig-uptimekuma.yaml
```
