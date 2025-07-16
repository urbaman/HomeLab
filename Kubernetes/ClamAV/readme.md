# ClamAV installation

Install the helm chart

```bash
helm repo add wiremind https://wiremind.github.io/wiremind-helm-charts
helm repo update
helm upgrade -i -n clamav clamav --create-namespace wiremind/clamav --set hpa.enabled=false
#helm upgrade -i -n clamav clamav --create-namespace wiremind/clamav --set resources.limits.cpu=200m --set resources.limits.memory=1536Mi --set resources.requests.cpu=100m --set resources.requests.memory=1024Mi --set hpa.cpu=80 --set hpa.memory=80
```

## Check from a client

```bash
sudo apt install -y clamdscan
sudo apt remove -y clamav-daemon clamav-freshclam
```

Port-forward the service

```bash
kubectl port-forward -n clamav service/clamav :3310
```

Add the option `TCPAddr` and `TCPSocket` (`TCPAddr 1.2.3.4` `TCPSocket <forwarded port>`) and remove (LocalSocket) from the client on `/etc/clamav/clamd.conf` file (create if not present).

Create a file and scan it:

```bash
touch prova
clamdscan -v --stream [--config-file=/etc/clamav/clamd.conf] --fdpass prova
```
