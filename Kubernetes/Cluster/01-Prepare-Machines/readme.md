# Preparing the servers

## datetime

Check timedatectl status:

```bash
timedatectl status
```

If the Time zone is different from the wanted one, we can list all the available timezones:

```bash
timedatectl list-timezones
```

Then, set the timezone:

```bash
timedatectl set-timezone "Europe/Rome"
```

## postfix for sending mail

### Install packages

```bash
sudo apt install postfix mailutils libsasl2-modules postfix-pcre
```

### setup postfix logs

```bash
sudo touch /var/log/mail.log
sudo touch /var/log/mail.err
sudo touch /var/log/mail.warn
sudo touch /var/log/mail.info
sudo chown syslog:adm /var/log/mail*
```

Change rsyslog conf in /etc/rsyslog.d/50-default.conf, restart the service

```bash
#  Default rules for rsyslog.
#
#                       For more information see rsyslog.conf(5) and /etc/rsyslog.conf

#
# First some standard log files.  Log by facility.
#
auth,authpriv.*                 /var/log/auth.log
*.*;auth,authpriv.none          -/var/log/syslog
#cron.*                         /var/log/cron.log
#daemon.*                       -/var/log/daemon.log
kern.*                          -/var/log/kern.log
#lpr.*                          -/var/log/lpr.log
mail.*                          -/var/log/mail.log
#user.*                         -/var/log/user.log

#
# Logging for the mail system.  Split it up so that
# it is easy to write scripts to parse these files.
#
mail.info                       -/var/log/mail.info
mail.warn                       -/var/log/mail.warn
mail.err                        /var/log/mail.err

#
# Some "catch-all" log files.
#
#*.=debug;\
#       auth,authpriv.none;\
#       news.none;mail.none     -/var/log/debug
#*.=info;*.=notice;*.=warn;\
#       auth,authpriv.none;\
#       cron,daemon.none;\
#       mail,news.none          -/var/log/messages

#
# Emergencies are sent to everybody logged in.
#
*.emerg                         :omusrmsg:*

#
# I like to have messages displayed on the console, but only on a virtual
# console I usually leave idle.
#
#daemon,mail.*;\
#       news.=crit;news.=err;news.=notice;\
#       *.=debug;*.=info;\
#       *.=notice;*.=warn       /dev/tty8
```

```bash
sudo service rsyslog restart
```

### Configure postfix

Change /etc/postfix/main.cf to include/change these lines:

```bash
# See /usr/share/postfix/main.cf.dist for a commented, more complete version

myhostname=host.domain.com

smtpd_banner = $myhostname ESMTP $mail_name (Debian/GNU)
biff = no

# appending .domain is the MUA's job.
append_dot_mydomain = no

# Uncomment the next line to generate "delayed mail" warnings
#delay_warning_time = 4h

alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
#mydestination = $myhostname, localhost.$mydomain, localhost
#relayhost =
mynetworks = 127.0.0.0/8
inet_interfaces = loopback-only
recipient_delimiter = +

#Relay
relayhost = [smtp.domain.com]:587
smtp_use_tls = yes
smtp_sasl_auth_enable = yes
smtp_sasl_security_options = noanonymous
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt

compatibility_level = 2

smtp_header_checks = pcre:/etc/postfix/smtp_header_checks
```

Be sure there are no dupes as the main.cf may have smtp_sasl_security_options = {} , and relayhost = {}. Just delete or comment those lines.

Create an /etc/postfix/sasl_passwd file with:

```bash
[smtp.domain.com]:587    testmehere@gmail.com:PASSWD
```

run

```bash
sudo chmod 600 /etc/postfix/sasl_passwd
sudo postmap /etc/postfix/sasl_passwd
```

Set the default root email in /etc/aliases adding the following line:

```bash
root me@myemail.com
```

```bash
sudo newaliases
```

#### Customize From field

Create /etc/postfix/smtp_header_checks file, this changes all outgoing mail:

```bash
/^From:.*/ REPLACE From: HOSTNAME-alert <HOSTNAME-alert@something.com>
```

```bash
sudo chmod 600 /etc/postfix/smtp_header_checks
sudo postmap /etc/postfix/smtp_header_checks
```

Restart service:

```bash
service postfix restart
```

### Test

```bash
echo "Test mail from postfix" | mail -s "Test Postfix" test@test.com
```

Logs:

```bash
/var/log/mail.warn
```

is helpful as well as

```bash
/var/log/mail.info
```

## Enable automatic updates

### Install unattended-upgrades package

As discussed before, the first step is to install the unattended-upgrades package. To achieve this, we will use the APT package manager as follows:

```bash
sudo apt install unattended-upgrades
```

When the installation is complete, verify using the following systemctl command:

```bash
sudo systemctl status unattended-upgrades
```

By default, the unattended-upgrades daemon should run once the installation is complete as evidenced in the output below.

```bash
Unit unattended-updates.service could not be found.
ubuntu@k8cp1:~$ service unattended-upgrades status
● unattended-upgrades.service - Unattended Upgrades Shutdown
     Loaded: loaded (/lib/systemd/system/unattended-upgrades.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2022-07-17 00:48:30 CEST; 1 day 11h ago
       Docs: man:unattended-upgrade(8)
   Main PID: 922 (unattended-upgr)
      Tasks: 2 (limit: 38453)
     Memory: 10.9M
        CPU: 151ms
     CGroup: /system.slice/unattended-upgrades.service
             └─922 /usr/bin/python3 /usr/share/unattended-upgrades/unattended-upgrade-shutdown --wait-for-signal
```

To set automatic updates, we are going to install the update-notifier-common package.:

```bash
sudo apt install update-notifier-common
```

### Configure unattended-upgrades service

In this step, we are going to make changes to the unattended-upgrades configuration file.

```bash
sudo vi /etc/apt/apt.conf.d/50unattended-upgrades
```

The file helps you to specify which packages should automatically be updated or skipped during the update process. By default, however, only security updates are set to be automatically installed as shown in the lines below. Therefore, no action is needed if we just need the security updates.
We enabled also normal updates and kubernetes and docker repositories:

```bash
// Automatically upgrade packages from these (origin:archive) pairs
//
// Note that in Ubuntu security updates may pull in new dependencies
// from non-security sources (e.g. chromium). By allowing the release
// pocket these get automatically pulled in.
Unattended-Upgrade::Allowed-Origins {
        "${distro_id}:${distro_codename}";
        "${distro_id}:${distro_codename}-security";
        // Extended Security Maintenance; doesn't necessarily exist for
        // every release and this system may not have it installed, but if
        // available, the policy for updates is such that unattended-upgrades
        // should also install from here by default.
        "${distro_id}ESMApps:${distro_codename}-apps-security";
        "${distro_id}ESM:${distro_codename}-infra-security";
        "${distro_id}:${distro_codename}-updates";
        "Docker:${distro_codename}";
        "kubernetes-xenial:kubernetes-xenial";
//      "${distro_id}:${distro_codename}-proposed";
//      "${distro_id}:${distro_codename}-backports";
};
```

To see how to define the repositories, run

```bash
sudo apt-cache policy
```

```bash
Package files:
 100 /var/lib/dpkg/status
     release a=now
 500 https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
     release o=kubernetes-xenial,a=kubernetes-xenial,n=kubernetes-xenial,l=kubernetes-xenial,c=main,b=amd64
     origin apt.kubernetes.io
 500 https://download.docker.com/linux/ubuntu jammy/stable amd64 Packages
     release o=Docker,a=jammy,l=Docker CE,c=stable,b=amd64
     origin download.docker.com
```

In the output, find the o=xxx and a=yyy values, and add them in the list above separated by a semicolon. We also see that the baltocdn repository (for the helm package) does not have an origin set, so it is not usable in unattended updates.

Line starting with double slashes ( // ) are commented. If you want to update a repository you need to uncomment or remove the double slash signs.

For example, to blacklist some packages from being upgraded, remove the double slash signs in the line with the parameter Unattended-Upgrade::Package-Blacklist {

Then specify the package names. In the example below, we have prevented the kubectl, kubelet and kubeadm packages from being upgraded.

```bash
// Python regular expressions, matching packages to exclude from upgrading
Unattended-Upgrade::Package-Blacklist {
    // The following matches all packages starting with linux-
//  "linux-";

    // Use $ to explicitely define the end of a package name. Without
    // the $, "libc6" would match all of them.
//  "libc6$";
//  "libc6-dev$";
//  "libc6-i686$";

    // Special characters need escaping
//  "libstdc\+\+6$";

    // The following matches packages like xen-system-amd64, xen-utils-4.1,
    // xenstore-utils and libxenstore3.0
//  "(lib)?xen(store)?";

    // For more information about Python regular expressions, see
    // https://docs.python.org/3/howto/regex.html
"kubelet"
"kubectl"
"kubeadm"
};
```

When you scroll down, you can see a host of other options that you might decide to enable or leave them as they are.

### Enable email notifications

Sometimes, you may want to receive email notifications. To achieve this, scroll and locate the line below and remove the preceding double slashes.

```bash
//Unattended-Upgrade::Mail " ";
```

Be sure to specify the recipient email address.

```bash
Unattended-Upgrade::Mail "me@example.com ";
```

In addition, you can choose to receive email updates in case an update goes wrong, such as when security updates fail. To do so, locate this line:

```bash
//Unattended-Upgrade::MailReport  "on-change";
```

uncomment it and change the attribute "on-change" to "only-on-error"

When security updates are installed, it's always good practice to restart the server in order to update the kernel. You can enable an automatic reboot by locating the line below. I  our implementation, we do not enable this feature.

```bash
//Unattended-Upgrade::Automatic-Reboot "false";
```

Change the "false" value to "true"

If there are users logged in and you would desire to proceed with the reboot, locate the line"

```bash
// Unattended-Upgrade::Automatic-Reboot-WithUsers "true";
```

You can also determine the time the update will occur by uncommenting the line below. By default, this is set to 4:00 am.

```bash
// Unattended-Upgrade::Automatic-Reboot-Time "04:00";
```

For this specific implementation, we will set unattended-upgrades to automatic reboot on etcd and haproxy nodes, setting the automatic reboot time with half an hour offset from each other, starting at 02:00 for the first node and finishing at 04:30 for the last node.

There are many other rules you can set to suit your needs. Simply scroll and uncomment the directives as we have just elaborated.

Once you are done, save the changes and exit the configuration file. That's about it in this section.

### Enable automatic updates on Ubuntu 22.04

Finally, to enable automatic upgrades , edit the 20auto-upgrades file as shown.

```bash
sudo vi /etc/apt/apt.conf.d/20auto-upgrades
```

By default, the file has two lines as shown.

```bash
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
```

These lines allow you to determine how the upgrade will occur. The first line handles the update of the package lists while the second one initiates the automatic upgrades.

The value "1" enables the auto-update and the auto-upgrade respectively. If you want to disable it, set this value to "0".

Finally, restart unattended-upgrades service

```bash
sudo service unattended-upgrades restart
```
