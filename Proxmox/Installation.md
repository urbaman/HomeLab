# Proxmox installation

I choose to install Proxmox in ZFS Raid 1 mirror on the 2 Crucial MX500 500 GB SATA SSD drives connected directly to the SUPERMICRO X10SRi-F motherboard. I did not want to occupy the whole drives but only use a 64 GB partition. The rest of the SSD drives will be used to run the VMs.

I downloaded the latest ISO Installer and flashed it to an USB drive using Etcher.

Booting your server from the USB drive, you’ll get the welcome screen. Select the first option.

On the next screen, select zfs (Raid 1). Then click Deselect All and select as Harddisk  0 and Harddisk 1 both internaly installed SSD’s.

Click on Advanced Options.

Set the compression to lz4. Then reduce the hdsize to about 80% of the SSD size. This way you enable overprovisioning. In my case, I use 400 GB of the available 500 GB.

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

apt install ifupdown2 -y

## Activating network interfaces

In case your motherboard has multiple NICs, not all network interfaces will be activated by default. The primary network interface, the one specified during installation, will of course be activated. All other NICs will stay deactivated in case they don't recieve an IP address during boot time.

Log in to Proxmox and browse to your PVE. Then select System → Network and double-click a deactivated network device. Enter an valid IP address and click the Autostart option. Then click OK. Do the same for all other deactivated NICs. Then click the Apply Configuration button.

## Configuring E-mail alerts using GMail

Follow [[this excellent article](https://geekistheway.com/2021/03/07/configuring-e-mail-alerts-on-your-proxmox/)] on how to set up e-mail alerts on your Proxmox VE server.

### Configuring E-mail alerts using another e-mail provider 

Log in to Proxmox and browse to your PVE. Then select Shell.

Install the required packages :

apt update && apt install -y libsasl2-modules
Configure a relay account :

echo "[mailserver]:port username:password" > /etc/postfix/sasl_passwd
Replace mailserver, port, username and password with your own values.

Create a hash from the sasl_passwd file :

postmap /etc/postfix/sasl_passwd
Secure your sasl_passwd files :

chown root:root /etc/postfix/sasl_passwd*
chmod 0600 /etc/postfix/sasl_passwd*
Configure postfix to use the relay account :

nano /etc/postfix/main.cf
Delete the line :

relayhost =
Add the lines :

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
Restart the postfix service :

systemctl restart postfix
Test your new configuration :

echo "My first test message" | mail -s "Proxmox alert test" somebody@somewhere.com
