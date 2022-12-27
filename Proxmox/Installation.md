# Proxmox installation

I choose to install Proxmox in ZFS Raid 1 mirror on the 2 Samsung 960 GB Nvme SSD drives on the front bay.

I downloaded the latest ISO Installer, started the IDRAC virtual console, enabled the virtual media and added the ISO to the virtual CD-ROM, and it booted from there.

On the next screen, select zfs (Raid 1). Then click Deselect All and select as Harddisk 0 and Harddisk 1 both Samsung Nvme SSDs.

Click on Advanced Options.

Click OK and then Next.

Select your Country, Time zone and Keyboard Layout.

Then click Next.

Enter a strong password and confirm it.

Enter the system administrators email address. It will be used to send you warnings and maintenance messages.

Click on Next.

Select the NIC to be used by Proxmox VE , set the hostname and give your system a fixed IP address inside your LAN IP range.

Click on Next.

Verify your settings in the summary screen.

Either go back and correct some settings or click Install to continue.

Proxmox VE will be installed on both SSDs.

Take note of the mentioned URL. It’s the URL you’ll use to remotely manage your new Proxmox VE installation.

Click Reboot.

Remove the USB drive during reboot.

When all goes well, the system will reboot and start Proxmox VE. If not, see the section PROXMOX Boot problem below.

# Proxmox Log in

Open a browser and go to the URL mentioned during the installation process.

You’ll get a security warning. Accept it and continue.

Log in to the GUI as user root and use the password set during the installation procedure.

Click Login.

You’ll see your datacenter welcome screen showing the installed Proxmox VE node.

Click on your Proxmox VE node to check its current status.

# Post installation

## Enabling applying network configuration without reboot

Log in to Proxmox and browse to your PVE. Then select Shell.

Enther the command : 

```bash
apt install ifupdown2 -y
```

## Activating network interfaces

In case your motherboard has multiple NICs, not all network interfaces will be activated by default. The primary network interface, the one specified during installation, will of course be activated. All other NICs will stay deactivated in case they don't recieve an IP address during boot time.

Log in to Proxmox and browse to your PVE. Then select System → Network and double-click a deactivated network device. Enter an valid IP address and click the Autostart option. Then click OK. Do the same for all other deactivated NICs. Then click the Apply Configuration button.

## Configuring E-mail alerts using GMail

Follow [this excellent article](https://geekistheway.com/2021/03/07/configuring-e-mail-alerts-on-your-proxmox/) on how to set up e-mail alerts on your Proxmox VE server.

### Configuring E-mail alerts using another e-mail provider 

Log in to Proxmox and browse to your PVE. Then select Shell.

Install the required packages :
```bash
apt update && apt install -y libsasl2-modules
```

Configure a relay account :

```bash
echo "[mailserver]:port username:password" > /etc/postfix/sasl_passwd
```

Replace mailserver, port, username and password with your own values.

Create a hash from the sasl_passwd file :

```bash
postmap /etc/postfix/sasl_passwd
```

Secure your sasl_passwd files :

```bash
chown root:root /etc/postfix/sasl_passwd*
chmod 0600 /etc/postfix/sasl_passwd*
```

Configure postfix to use the relay account :

```bash
nano /etc/postfix/main.cf
```

Delete the line :

```bash
relayhost =
```

Add the lines :

```bash
relayhost = [mailserver.somecompany.com]:587
# enable SASL authentication
smtp_sasl_auth_enable = yes
# disallow methods that allow anonymous authentication.
smtp_sasl_security_options = noanonymous
# where to find sasl_passwd
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
# Enable STARTTLS encryption
smtp_use_tls = yes
# where to find CA certificates
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt
```

Restart the postfix service :

```bash
systemctl restart postfix
```

Test your new configuration :

```bash
echo "My first test message" | mail -s "Proxmox alert test" somebody@somewhere.com
```

Customize the from header:

```bash
apt install postfix-pcre
```

```bash
nano /etc/postfix/smtp_header_checks
```

```bash
/^From:.*/ REPLACE From: HOSTNAME-alert <HOSTNAME-alert@something.com>
```

Create a hash from the smtp_header_checks file

```bash
postmap /etc/postfix/smtp_header_checks
```

Secure your sasl_passwd files :

```bash
chown root:root /etc/postfix/smtp_header_checks*
chmod 0600 /etc/postfix/smtp_header_checks*
```

Add to main.cf:

```bash
smtp_header_checks = pcre:/etc/postfix/smtp_header_checks
```

```bash
systemctl restart postfix
```

Test your new configuration :

```bash
echo "My first test message" | mail -s "Proxmox alert test" somebody@somewhere.com
```
