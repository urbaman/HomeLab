# Certbot Let's Encrypt with Cloudflare

First of all, create a token from cloudflare to modify the domain associated with your server's FQDN (for example host.domain.com).

Finally, install certbot with cloudflare plugin, create the credentials file, make it safe and launch the certificate request, answering the few questions:

```bash
sudo apt install certbot python3-certbot-dns-cloudflare -y
sudo mkdir /etc/certbot
sudo vi /etc/certbot/credentials
```

```bash
dns_cloudflare_api_token = abcdef0123456789abcdef0123456789abcdef01
```

```bash
sudo chmod 600 /etc/certbot/credentials
sudo certbot certonly \
  --dns-cloudflare \
  --dns-cloudflare-credentials /etc/certbot/credentials \
  -d host.domain.com
```

We are ready to modify apache's vhost config file

```bash
vi /etc/apache2/sites-available/host.domain.com
```

```bash
SSLEngine On
   SSLCertificateFile /etc/letsencrypt/live/host.domain.com/fullchain.pem
   SSLCertificateKeyFile /etc/letsencrypt/live/host.domain.com/privkey.pem
    # If you have either an SSLCertificateChainFile or, a self-signed CA signed certificate
    # uncomment the line below.
    # Note: Some versions of Apache will not accept the SSLCertificateChainFile directive.
    # Try using SSLCACertificateFile instead in that case.
    # SSLCertificateChainFile /etc/ssl/certs/landscape_server_ca.crt
    # Disable to avoid POODLE attack
    SSLProtocol all -SSLv3 -SSLv2 -TLSv1
    SSLHonorCipherOrder On
    SSLCompression Off
    SSLCipherSuite EECDH+AESGCM+AES128:EDH+AESGCM+AES128:EECDH+AES128:EDH+AES128:ECDH+AESGCM+AES128:aRSA+AESGCM+AES128:ECDH+AES128:DH+AES128:aRSA+AES128:EECDH+AESGCM:EDH+AESGCM:EECDH:EDH:ECDH+AESGCM:aRSA+AESGCM:ECDH:DH:aRSA:HIGH:!MEDIUM:!aNULL:!NULL:!LOW:!3DES:!DSS:!EXP:!PSK:!SRP:!CAMELLIA:!DHE-RSA-AES128-SHA:!DHE-RSA-AES256-SHA:!aECDH
```

Restart apache2.
