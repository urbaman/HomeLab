# Groundcover for Observability

## Installation

**NB**: beware that you will need two storage volumes, 128Gi and 100Gi big.

Go to [https://www.groundcover.com/](https://www.groundcover.com/), register and run the proposed command on a control plane, then wait for the deployment to finish.

Exit the ssh session end login again to enable the groundcover command.

## Data obfuscation.

Because Groundcover sends data to the cloud, you can set the data to be obfuscated. Create a groundcover-obfuscate.yaml file with the following content

```yaml
agent:
  alligator:
    obfuscateData: true
```

Then, deploy with the groundcover commad, beware that it will ask you to authenticate with a browser.

```bash
groundcover deploy --values groundcover-obfuscate.yaml
```
