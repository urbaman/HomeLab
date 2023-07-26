# Homer dashboard installation

## Preparation

Create a Longhorn Volume, and the required homer.domain.com  DNS entry (we use cloudflare)

## Deploy

Run the yaml file to deploy Homer.

```bash
kubectl apply -f homer.yaml
```

## Config

Run `kubectl cp` to put logos into the assets/tools directory inside the pod

```bash
kubectl cp logo.png namespace/pod:/www/assets/tools
```

Run kubectl exec to get inside the pod console and edit the config, to add apps and whatever.

```bash
kubectl exec -it -n namespace pod -- /bin/sh
/www $ vi assets/config.yml
```

You can also use `kubectl cp` to extract the config.yml file, edit it outside of the pod, and copy it back in.

```bash
kubectl cp namespace/pod:/www/assets/config.yml /full/local/path
```
