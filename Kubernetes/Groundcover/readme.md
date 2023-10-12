# Groundcover for Observability

## Installation

**NB**: beware that you will need two storage volumes, 128Gi and 100Gi big.

Go to [https://www.groundcover.com/](https://www.groundcover.com/), register and run `groundcover deploy --token <TOKEN> --values groundcover-values.yaml` on a control plane, then wait for the deployment to finish.

Exit the ssh session end login again to enable the groundcover command.

## Modify the deployment, examples

### Data obfuscation

Because Groundcover sends data to the cloud, you can set the data to be obfuscated. Create a groundcover-values.yaml file with the following content

```yaml
agent:
  alligator:
    obfuscateData: true
```

### Storage Class

Add the storage class to the groundcover-values.yaml file

```yaml
clickhouse:
  # logs storage
  persistence:
    storageClass: longhorn-nvme
    size: 128Gi

victoria-metrics-single:
  # metrics storage
  server:
    persistentVolume:
      storageClass: longhorn-nvme
      size: 100Gi 
```

Then, delete the deploy and re-deploy with the groundcover commad, beware that it will ask you to authenticate with a browser.

```bash
groundcover delete
groundcover deploy -f groundcover-values.yaml
```
