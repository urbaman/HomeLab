# Kubernetes

## Cluster structure

- 3x controllers: k8cp1, k8cp2, k8cp3
- 3x workers: k8w1, k8w2, k8w3
- 3x etcd cluster: etcd1, etce2, etcd3
- 3x haproxy+keepalive: haproxy1, haproxy2, haproxy3
- 3x glusterfs cluster: truenas1, truenas2, truenas3
- 1x glusterfs clusater manager: truecommand

| Hostname               | CPU | RAM  | Disk                     | System             | Role                              | IP         |
| ---------------------- | --- | ---- | ------------------------ | ------------------ | --------------------------------- | ---------- |
| truenas1.urbaman.it    | 4   | 16GB | 2x32GB (OS), 2x5TB (GFS) | TrueNAS SCALE      | GlusterFS node 1                  | 10.0.50.21 |
| truenas2.urbaman.it    | 4   | 16GB | 2x32GB (OS), 2x5TB (GFS) | TrueNAS SCALE      | GlusterFS node 2                  | 10.0.50.22 |
| truenas3.urbaman.it    | 4   | 16GB | 2x32GB (OS), 2x5TB (GFS) | TrueNAS SCALE      | GlusterFS node 3                  | 10.0.50.23 |
| truecommand.urbaman.it | 2   | 4GB  | 32GB                     | Debian/Truecommand | GlusterFS cluster manager         | 10.0.50.24 |
| k8cp1.urbaman.it       | 8   | 32GB | 16BG                     | Ubuntu 22.04       | Kubernetes control manager node 1 | 10.0.50.31 |
| k8cp2.urbaman.it       | 8   | 32GB | 16BG                     | Ubuntu 22.04       | Kubernetes control manager node 2 | 10.0.50.32 |
| k8cp3.urbaman.it       | 8   | 32GB | 16BG                     | Ubuntu 22.04       | Kubernetes control manager node 3 | 10.0.50.33 |
| k8w1.urbaman.it        | 8   | 32GB | 16BG                     | Ubuntu 22.04       | Kubernetes worker node 1          | 10.0.50.34 |
| k8w2.urbaman.it        | 8   | 32GB | 16BG                     | Ubuntu 22.04       | Kubernetes worker node 2          | 10.0.50.35 |
| k8w3.urbaman.it        | 8   | 32GB | 16BG                     | Ubuntu 22.04       | Kubernetes worker node 3          | 10.0.50.36 |
| ectd1.urbaman.it       | 2   | 4GB  | 16GB                     | Ubuntu 22.04       | Etcd cluster node 1               | 10.0.50.41 |
| etcd2.urbaman.it       | 2   | 4GB  | 16GB                     | Ubuntu 22.04       | Etcd cluster node 2               | 10.0.50.42 |
| etcd3.urbaman.it       | 2   | 4GB  | 16GB                     | Ubuntu 22.04       | Etcd cluster node 3               | 10.0.50.43 |
| haproxy1.urbaman.it    | 2   | 4GB  | 16GB                     | Ubuntu 22.04       | Keepalive Master/Haproxy node 1   | 10.0.50.61 |
| haproxy2.urbaman.it    | 2   | 4GB  | 16GB                     | Ubuntu 22.04       | Keepalive Backup/Haproxy node 2   | 10.0.50.62 |
| haproxy3.urbaman.it    | 2   | 4GB  | 16GB                     | Ubuntu 22.04       | Keepalive Backup/Haproxy node 3   | 10.0.50.63 |
| k8cp.urbaman.it        | N/A | N/A  | N/A                      | N/A                | Keepalive VIP IP                  | 10.0.50.64 |

## 0) Preparing the servers

### datetime

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

### postfix for sending mail

#### Instll packages:

```bash
sudo apt install postfix mailutils libsasl2-modules postfix-pcre
```

#### setup postfix logs:

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

#### Configure postfix

Change /etc/postfix/main.cf to include/change these lines:

```bash
# See /usr/share/postfix/main.cf.dist for a commented, more complete version

myhostname=k8cp1.urbaman.it

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
relayhost = [smtp.urbaman.it]:587
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
[smtp.gmail.com]:587    testmehere@gmail.com:PASSWD
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

##### Customize From field

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

Test:

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

### Enable automatic updates

#### Install unattended-upgrades package

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

#### Configure unattended-upgrades service

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
 500 https://baltocdn.com/helm/stable/debian all/main amd64 Packages
     release a=all,n=all,c=main,b=amd64
     origin baltocdn.com
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

#### Enable email notifications

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

There are many other rules you can set to suit your needs. Simply scroll and uncomment the directives as we have just elaborated.

Once you are done, save the changes and exit the configuration file. That's about it in this section.

#### Enable automatic updates on Ubuntu 22.04

Finally, to enable automatic upgrades , edit the 20auto-upgrades file as shown.

sudo vi /etc/apt/apt.conf.d/20auto-upgrades

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

## 1) External Etcd

### Downloading etcd Binaries

On each Linux host, run the below commands to download and install the latest version of the binaries:

```bash
ETCD_VER=v3.5.4
DOWNLOAD_URL=https://storage.googleapis.com/etcd

rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
rm -rf /tmp/etcd-download-test
mkdir -p /tmp/etcd-download-test
curl -L ${DOWNLOAD_URL}/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz -o /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
tar xzvf /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz -C /tmp/etcd-download-test --strip-components=1
rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
chmod +x /tmp/etcd-download-test/etcd
chmod +x /tmp/etcd-download-test/etcdctl 

#Verify the downloads
/tmp/etcd-download-test/etcd --version
/tmp/etcd-download-test/etcdctl version

#Move them to the bin folder
sudo mv /tmp/etcd-download-test/etcd /usr/local/bin
sudo mv /tmp/etcd-download-test/etcdctl /usr/local/bin
sudo mv /tmp/etcd-download-test/etcdutl /usr/local/bin
```

### Generating and Distributing the Certificates

We will use cfssl tool from Cloudflare to generate the certificates and keys.

From a linux workstation:

```bash
apt install golang-cfssl 
```

Make a directory called certs and run the below commands to generate the CA certificate and server certificate and key combinations for each host.

```bash
mkdir certs
cd certs
```

First, let’s create the CA certificate that’s going to be used by all the etcd servers and the clients.

```bash
echo '{"CN":"CA","key":{"algo":"rsa","size":2048}}' | cfssl gencert -initca - | cfssljson -bare ca -
echo '{"signing":{"default":{"expiry":"43800h","usages":["signing","key encipherment","server auth","client auth"]}}}' > ca-config.json
```

This results in three files – ca-key.pem, ca.pem, and ca.csr

Next, we will generate the certificate and key for the first node.

```bash
export NAME=node-1
export ADDRESS=10.0.0.60,$NAME
echo '{"CN":"'$NAME'","hosts":[""],"key":{"algo":"rsa","size":2048}}' | cfssl gencert -config=ca-config.json -ca=ca.pem -ca-key=ca-key.pem -hostname="$ADDRESS" - | cfssljson -bare $NAME
```

Repeat the steps for the next two nodes.

```bash
export NAME=node-2
export ADDRESS=10.0.0.61,$NAME
echo '{"CN":"'$NAME'","hosts":[""],"key":{"algo":"rsa","size":2048}}' | cfssl gencert -config=ca-config.json -ca=ca.pem -ca-key=ca-key.pem -hostname="$ADDRESS" - | cfssljson -bare $NAME
```

```bash
export NAME=node-3
export ADDRESS=10.0.0.62,$NAME
echo '{"CN":"'$NAME'","hosts":[""],"key":{"algo":"rsa","size":2048}}' | cfssl gencert -config=ca-config.json -ca=ca.pem -ca-key=ca-key.pem -hostname="$ADDRESS" - | cfssljson -bare $NAME
```

At this point, we have the certificates and keys generated for the CA and all the three nodes.

It’s time to distribute these certificates to each node of the cluster.

Run the below command to copy the certificates to the respective nodes by replacing the username and the IP address.

```bash
HOST=10.0.0.60
USER=ubuntu

scp ca.pem $USER@$HOST:etcd-ca.crt
scp node-1.pem $USER@$HOST:server.crt
scp node-1-key.pem $USER@$HOST:server.key
```

SSH into each node and run the below commands to move the certificates into an appropriate directory.

```bash
HOST=10.0.0.60
USER=ubuntu

ssh $USER@$HOST
sudo mkdir -p /etc/etcd
sudo mv * /etc/etcd
sudo chmod 600 /etc/etcd/server.key
```

We are done with the generation and distribution of certificates on each node. In the next step, we will create the configuration file and the Systemd unit file per each node.

### Configuring and Starting the etcd Cluster

On node 1, create a file called etcd.conf in /etc/etcd directory with the below contents:

```bash
ETCD_NAME=node-1
ETCD_LISTEN_PEER_URLS="https://10.0.0.60:2380"
ETCD_LISTEN_CLIENT_URLS="https://10.0.0.60:2379"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER="node-1=https://10.0.0.60:2380,node-2=https://10.0.0.61:2380,node-3=https://10.0.0.62:2380"
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://10.0.0.60:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://10.0.0.60:2379"
ETCD_TRUSTED_CA_FILE="/etc/etcd/etcd-ca.crt"
ETCD_CERT_FILE="/etc/etcd/server.crt"
ETCD_KEY_FILE="/etc/etcd/server.key"
ETCD_PEER_CLIENT_CERT_AUTH=true
ETCD_PEER_TRUSTED_CA_FILE="/etc/etcd/etcd-ca.crt"
ETCD_PEER_KEY_FILE="/etc/etcd/server.key"
ETCD_PEER_CERT_FILE="/etc/etcd/server.crt"
ETCD_DATA_DIR="/var/lib/etcd"
```

For node 2, use the below content.

```bash
ETCD_NAME=node-2
ETCD_LISTEN_PEER_URLS="https://10.0.0.61:2380"
ETCD_LISTEN_CLIENT_URLS="https://10.0.0.61:2379"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER="node-1=https://10.0.0.60:2380,node-2=https://10.0.0.61:2380,node-3=https://10.0.0.62:2380"
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://10.0.0.61:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://10.0.0.61:2379"
ETCD_TRUSTED_CA_FILE="/etc/etcd/etcd-ca.crt"
ETCD_CERT_FILE="/etc/etcd/server.crt"
ETCD_KEY_FILE="/etc/etcd/server.key"
ETCD_PEER_CLIENT_CERT_AUTH=true
ETCD_PEER_TRUSTED_CA_FILE="/etc/etcd/etcd-ca.crt"
ETCD_PEER_KEY_FILE="/etc/etcd/server.key"
ETCD_PEER_CERT_FILE="/etc/etcd/server.crt"
ETCD_DATA_DIR="/var/lib/etcd"
```

Finally, create the configuration file for the last node.

```bash
ETCD_NAME=node-3
ETCD_LISTEN_PEER_URLS="https://10.0.0.62:2380"
ETCD_LISTEN_CLIENT_URLS="https://10.0.0.62:2379"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER="node-1=https://10.0.0.60:2380,node-2=https://10.0.0.61:2380,node-3=https://10.0.0.62:2380"
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://10.0.0.62:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://10.0.0.62:2379"
ETCD_TRUSTED_CA_FILE="/etc/etcd/etcd-ca.crt"
ETCD_CERT_FILE="/etc/etcd/server.crt"
ETCD_KEY_FILE="/etc/etcd/server.key"
ETCD_PEER_CLIENT_CERT_AUTH=true
ETCD_PEER_TRUSTED_CA_FILE="/etc/etcd/etcd-ca.crt"
ETCD_PEER_KEY_FILE="/etc/etcd/server.key"
ETCD_PEER_CERT_FILE="/etc/etcd/server.crt"
ETCD_DATA_DIR="/var/lib/etcd"
```

With the configuration in place, it’s time for us to create the systemd unit file on each node.

Create the file, etcd.service at /lib/systemd/system with the below content:

```bash
[Unit]
Description=etcd key-value store
Documentation=https://github.com/etcd-io/etcd
After=network.target

[Service]
Type=notify
EnvironmentFile=/etc/etcd/etcd.conf
ExecStart=/usr/local/bin/etcd
Restart=always
RestartSec=10s
LimitNOFILE=40000

[Install]
WantedBy=multi-user.target
```

Since the configuration per node is moved into the dedicated file (/etc/etcd/etcd.conf), the unit file remains the same for all the nodes.

Now we mount glusterfs for external data persiostence.

```bash
sudo apt install glusterfs-client
sudo mkdir -p /var/lib/etcd
sudo vi /etc/fstab
```

Add

```bash
glusterserver1:/volname/etcd/etcd1 /var/lib/etcd glusterfs defaults,_netdev,backup-volfile-servers=glusterserver2:glusterserver3 0 0
```

We are now ready to start the service. Run the below command on each node to start the etcd cluster.

```bash
sudo systemctl daemon-reload
sudo systemctl enable etcd
sudo systemctl start etcd
```

Make sure that the etcd service is up and running with no errors.

```bash
sudo systemctl status etcd
```

### Testing and Validating the Cluster

SSH into one of the nodes to connect to the cluster through the etcdctl CLI.

```bash
etcdctl --endpoints https://10.0.0.60:2379 --cert /etc/etcd/server.crt --cacert /etc/etcd/etcd-ca.crt --key /etc/etcd/server.key put foo bar
```

We inserted a key into the etcd database. Let’s see if we can retrieve it.

```bash
etcdctl --endpoints https://10.0.0.60:2379 --cert /etc/etcd/server.crt --cacert /etc/etcd/etcd-ca.crt --key /etc/etcd/server.key get foo
```

Next, let’s use the API endpoint to check the health of the cluster.

```bash
curl --cacert /etc/etcd/etcd-ca.crt --cert /etc/etcd/server.crt --key /etc/etcd/server.key https://10.0.0.60:2379/health
```

Finally, let’s ensure that all the nodes are participating in the cluster.

```bash
etcdctl --endpoints https://10.0.0.60:2379 --cert /etc/etcd/server.crt --cacert /etc/etcd/etcd-ca.crt --key /etc/etcd/server.key member list
```
Congratulations! You now have a secure, distributed, highly available etcd cluster that’s ready for a production-grade K3s cluster environment.

## 2) kubernetes

### cgroups

Some systems will mount cgroup v1 and cgroup v2 by default, just in different locations. It can help to see where those are with:

```bash
grep ^cgroup /etc/mtab
```

Example output (on Ubuntu 20.04 LTS):

```bash
cgroup2 /sys/fs/cgroup/unified cgroup2 rw,nosuid,nodev,noexec,relatime 0 0
cgroup /sys/fs/cgroup/systemd cgroup rw,nosuid,nodev,noexec,relatime,xattr,name=systemd 0 0
cgroup /sys/fs/cgroup/cpu,cpuacct cgroup rw,nosuid,nodev,noexec,relatime,cpu,cpuacct 0 0
cgroup /sys/fs/cgroup/devices cgroup rw,nosuid,nodev,noexec,relatime,devices 0 0
cgroup /sys/fs/cgroup/memory cgroup rw,nosuid,nodev,noexec,relatime,memory 0 0
cgroup /sys/fs/cgroup/freezer cgroup rw,nosuid,nodev,noexec,relatime,freezer 0 0
cgroup /sys/fs/cgroup/rdma cgroup rw,nosuid,nodev,noexec,relatime,rdma 0 0
cgroup /sys/fs/cgroup/cpuset cgroup rw,nosuid,nodev,noexec,relatime,cpuset 0 0
cgroup /sys/fs/cgroup/blkio cgroup rw,nosuid,nodev,noexec,relatime,blkio 0 0
cgroup /sys/fs/cgroup/net_cls,net_prio cgroup rw,nosuid,nodev,noexec,relatime,net_cls,net_prio 0 0
cgroup /sys/fs/cgroup/pids cgroup rw,nosuid,nodev,noexec,relatime,pids 0 0
cgroup /sys/fs/cgroup/hugetlb cgroup rw,nosuid,nodev,noexec,relatime,hugetlb 0 0
cgroup /sys/fs/cgroup/perf_event cgroup rw,nosuid,nodev,noexec,relatime,perf_event 0 0
```

However, this does not strictly tell you if your system supports cgroup v2. As the other answer mentioned, grep cgroup /proc/filesystems is great for that.

```bash
grep cgroup /proc/filesystems
```

If your system supports cgroupv2, you would see:

```bash
nodev   cgroup
nodev   cgroup2
```

On a system with only cgroupv1, you would only see:

```bash
nodev   cgroup
```

### Disable swap

Turn swap off:

```bash
sudo swapoff -a
```

And prevents it from turning on after reboots by commenting it in the /etc/fstab file:

```bash
sudo vi /etc/fstab
```

### Containerd

Load the necessary modules for Containerd:

```bash
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

Setup the required kernel parameters:

```bash
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
```

#### Using Docker Repository

Docker offers the containerd containerd.io package in the .deb format through the Docker repository. So, install the required packages for containerd installation.

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release
```

##### Set up Docker Repository

Then, add the Docker’s GPG key to your system.

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

And then, add the Docker repository to the system by running the below command.

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list
```

##### Install Containerd

After adding the Docker’s repository, update the repository index.

```bash
sudo apt update
```

Then, install containerd using the apt command.

```bash
sudo apt install -y containerd.io
```

By now, the containerd service should be up and running.

```bash
sudo systemctl status containerd
```

Output:

```bash
● containerd.service - containerd container runtime
     Loaded: loaded (/lib/systemd/system/containerd.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2022-05-01 13:59:49 EDT; 18s ago
       Docs: https://containerd.io
    Process: 2182 ExecStartPre=/sbin/modprobe overlay (code=exited, status=0/SUCCESS)
   Main PID: 2183 (containerd)
      Tasks: 8
     Memory: 18.8M
        CPU: 63ms
     CGroup: /system.slice/containerd.service
             └─2183 /usr/local/bin/containerd

May 01 13:59:49 ubuntu-2204 containerd[2183]: time="2022-05-01T13:59:49.941358574-04:00" level=info msg=serving... address=/run/containerd/containerd.sock.ttrpc
May 01 13:59:49 ubuntu-2204 containerd[2183]: time="2022-05-01T13:59:49.941402747-04:00" level=info msg=serving... address=/run/containerd/containerd.sock
May 01 13:59:49 ubuntu-2204 systemd[1]: Started containerd container runtime.
```

##### Containerd configuration for Kubernetes

Containerd uses a configuration file config.toml for managing demons. For Kubernetes, you need to configure the Systemd group driver for runC.

```bash
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```

Set the cgroup driver for runc to systemd which is required for the kubelet. For more information on this config file see the containerd configuration docs here and also here.

At the end of this section in /etc/containerd/config.toml

```bash
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
...
```

Around line 112, change the value for SystemCgroup from false to true.

```bash
SystemdCgroup = true
```

Next, use the below command to restart the containerd service.

```bash
sudo systemctl restart containerd
```

### kubelet, kubeadm, kubectl

Update the apt package index and install packages needed to use the Kubernetes apt repository:

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
```

Download the Google Cloud public signing key:

```bash
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```

Add the Kubernetes apt repository:

```bash
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Update apt package index, install kubelet, kubeadm and kubectl, and pin their version:

```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

### High availability

Install both keepalived and haproxy

```bash
apt install keepalived haproxy
```

#### Keepalived

##### All

```bash
sudo sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/' /etc/sysctl.conf
sudo echo "net.ipv4.ip_nonlocal_bind = 1" >> /etc/sysctl.conf
sudo sysctl -p
```

/etc/keepalived/check_apiserver.sh

```bash
#!/bin/sh

errorExit() {
    echo "*** $*" 1>&2
    exit 1
}

curl --silent --max-time 2 --insecure https://localhost:6443/ -o /dev/null || errorExit "Error GET https://localhost:6443/"
if ip addr | grep -q 10.0.50.64; then
    curl --silent --max-time 2 --insecure https://10.0.50.64:6443/ -o /dev/null || errorExit "Error GET https://10.0.50.64:6443/"
fi
```

##### Master

```bash
! /etc/keepalived/keepalived.conf
! Configuration File for keepalived
global_defs {
    router_id LVS_DEVEL
}
vrrp_script check_apiserver {
  script "/etc/keepalived/check_apiserver.sh"
  interval 3
  weight -2
  fall 10
  rise 2
}

vrrp_instance VI_1 {
    state MASTER
    interface ens18
    virtual_router_id 51
    priority 101
    authentication {
        auth_type PASS
        auth_pass 42
    }
    virtual_ipaddress {
        10.0.50.64
    }
    track_script {
        check_apiserver
    }
}
```

##### Backup

```bash
! /etc/keepalived/keepalived.conf
! Configuration File for keepalived
global_defs {
    router_id LVS_DEVEL
}
vrrp_script check_apiserver {
  script "/etc/keepalived/check_apiserver.sh"
  interval 3
  weight -2
  fall 10
  rise 2
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens18
    virtual_router_id 51
    priority 100
    authentication {
        auth_type PASS
        auth_pass 42
    }
    virtual_ipaddress {
        10.0.50.64
    }
    track_script {
        check_apiserver
    }
}
```

##### All

```bash
sudo service keepalived restart
```

#### Haproxy

##### All

```bash
# /etc/haproxy/haproxy.cfg
#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    log /dev/log local0
    log /dev/log local1 notice
    daemon

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 1
    timeout http-request    10s
    timeout queue           20s
    timeout connect         5s
    timeout client          20s
    timeout server          20s
    timeout http-keep-alive 10s
    timeout check           10s

#---------------------------------------------------------------------
# apiserver frontend which proxys to the control plane nodes
#---------------------------------------------------------------------
frontend apiserver
    bind *:6443
    mode tcp
    option tcplog
    default_backend apiserver

#---------------------------------------------------------------------
# round robin balancing for apiserver
#---------------------------------------------------------------------
backend apiserver
    option httpchk GET /healthz
    http-check expect status 200
    mode tcp
    option ssl-hello-chk
    balance     roundrobin
        server k8cp1 k8cp1.urbaman.it:6443 check
        server k8cp2 k8cp2.urbaman.it:6443 check
        server k8cp3 k8cp3.urbaman.it:6443 check
```

Restart haproxy

```bash
sudo service haproxy restart
```

And check connection from another machine

```bash
nc -v 10.0.50.64 6443
```

### Set up the etcd cluster

Follow these instructions to set up the etcd cluster.

Copy the following files from any etcd node in the etcd cluster to the first control panel node:
Copy etcd certs to first control panel

```bash
export CONTROL_PLANE="ubuntu@10.0.50.31"
scp /etc/etcd/etcd-ca.crt "${CONTROL_PLANE}":/home/ubuntu/ca.crt
scp /etc/etcd/server.crt "${CONTROL_PLANE}":/home/ubuntu/apiserver-etcd-client.crt
scp /etc/etcd/server.key "${CONTROL_PLANE}":/home/ubuntu/apiserver-etcd-client.key
```

Replace the value of CONTROL_PLANE with the user@host of the first control-plane node.

On the first control panel, copy the files to the following locations:

```bash
sudo cp ca.crt /etc/kubernetes/pki/etcd/
sudo cp apiserver-etcd-client.crt /etc/kubernetes/pki/
sudo cp apiserver-etcd-client.key /etc/kubernetes/pki/
```

### Set up the first control plane node

Create a file called kubeadm-config.yaml with the following contents (modify IPs and hostnames as needed, then eliminate comments):

```bash
---
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
kubernetesVersion: stable
controlPlaneEndpoint: "k8cp.urbaman.it:6443" # change this (see below)
networking:
  podSubnet: "10.0.80.0/24" # change this (see below)
etcd:
  external:
    endpoints:
      - https://10.0.50.41:2379 # change ETCD_0_IP appropriately
      - https://10.0.50.42:2379 # change ETCD_1_IP appropriately
      - https://10.0.50.43:2379 # change ETCD_2_IP appropriately
    caFile: /etc/kubernetes/pki/etcd/ca.crt
    certFile: /etc/kubernetes/pki/apiserver-etcd-client.crt
    keyFile: /etc/kubernetes/pki/apiserver-etcd-client.key
```

Note: The difference between stacked etcd and external etcd here is that the external etcd setup requires a configuration file with the etcd endpoints under the external object for etcd. In the case of the stacked etcd topology, this is managed automatically.

Remember to change the controlPlaneEndpoint, podSubnet (equal to following calico IP CIDR), etcd endpoints (equal to the endpoints of the external etcd)

The following steps are similar to the stacked etcd setup:

```bash
sudo kubeadm init --config kubeadm-config.yaml --upload-certs
```

Write the output join commands that are returned to a text file for later use.

### Apply the CNI plugin of your choice.

Note: You must pick a network plugin that suits your use case and deploy it before you move on to next step. If you don't do this, you will not be able to launch your cluster properly. We will use Calico:

```bash
kubectl create -f https://docs.projectcalico.org/manifests/tigera-operator.yaml 
curl https://projectcalico.docs.tigera.io/manifests/custom-resources.yaml -O
```

If you wish to customize the Calico install, customize the downloaded custom-resources.yaml manifest locally (for example, customizing the IP CIDR).

```bash
kubectl create -f custom-resources.yaml
```

The, intstall the calicoctl to manage Calico.

```bash
kubectl apply -f https://projectcalico.docs.tigera.io/manifests/calicoctl.yaml
alias calicoctl="kubectl exec -i -n kube-system calicoctl -- /calicoctl" 
```

This way, in order to use the calicoctl alias when reading manifests, redirect the file into stdin, for example:

```bash
calicoctl create -f - < my_manifest.yaml
```

### Setup kubectl for a normal user

To start using your cluster, you need to run the following as a regular user:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Steps for the rest of the control panel nodes

The steps are the same as for the stacked etcd setup:

Make sure the first control plane node is fully initialized.

Join each control plane node with the join command you saved to a text file. It's recommended to join the control plane nodes one at a time.
Don't forget that the decryption key from --certificate-key expires after two hours, by default.

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Install workers

Worker nodes can be joined to the cluster with the command you stored previously as the output from the kubeadm init command:

```bash
sudo kubeadm join 192.168.0.200:6443 --token 9vr73a.a8uxyaju799qwdjv --discovery-token-ca-cert-hash sha256:7c2e69131a36ae2a042a339b33381c6d0d43887e2de83720eff5359e26aec866
```

From one of the control planes:

```bash
sudo scp /etc/kubernetes/admin.conf ubuntu@k8w1:/home/ubuntu
sudo scp /etc/kubernetes/admin.conf ubuntu@k8w2:/home/ubuntu
sudo scp /etc/kubernetes/admin.conf ubuntu@k8w3:/home/ubuntu
```

```bash
mkdir -p $HOME/.kube
sudo cp -i admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Control panel node isolation

By default, your cluster will not schedule Pods on the control plane nodes for security reasons. If you want to be able to schedule Pods on the control plane nodes, for example for a single machine Kubernetes cluster, run:

```bash
kubectl taint nodes --all node-role.kubernetes.io/control-plane- node-role.kubernetes.io/master-
kubectl taint nodes --all node-role.kubernetes.io/control-plane:NoSchedule-
```
