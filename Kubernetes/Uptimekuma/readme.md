# Uptime Kuma for service uptime monitoring

## Install Uptime Kuma through helm

Once Helm has been set up correctly, add the repo as follows:

```bash
helm repo add uptime-kuma https://dirsigler.github.io/uptime-kuma-helm
```

If you had already added this repo earlier, run helm repo update to retrieve the latest versions of the packages. You can then run helm search repo uptime-kuma to see the charts.

Get the values and change the version to latest:

```yaml
image:
  pullPolicy: IfNotPresent
  repository: louislam/uptime-kuma
  tag: latest
```

To install the uptime-kuma chart:

```bash
helm upgrade my-uptime-kuma uptime-kuma/uptime-kuma --install --namespace monitoring --values ukuma.yaml
```

## Expose through IngressRoute

Create and install the ssl cert and ingressroute

```bash
kubectl apply -f ir-uptimekuma.yaml
```
