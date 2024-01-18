# ClamAV installation

Install the helm chart

```bash
helm repo add wiremind https://wiremind.github.io/wiremind-helm-charts
helm repo update
helm update -i -n clamav --create-namespace wiremind/clamav
```

Edit the Statefulset, setting after the volumemounts

```yaml
      dnsConfig:
        options:
          - name: ndots
            value: "1"
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
