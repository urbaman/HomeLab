# External Etcd

## Downloading etcd Binaries

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

## Generating and Distributing the Certificates

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
export NAME=etcd1
export ADDRESS=10.0.50.41,$NAME,10.0.50.64,k8cp,k8cp.urbaman.it,10.0.50.42,etcd2,10.0.50.43,etcd3
echo '{"CN":"'$NAME'","hosts":[""],"key":{"algo":"rsa","size":2048}}' | cfssl gencert -config=ca-config.json -ca=ca.pem -ca-key=ca-key.pem -hostname="$ADDRESS" - | cfssljson -bare $NAME
```

Repeat the steps for the next two nodes.

```bash
export NAME=etcd2
export ADDRESS=10.0.50.42,$NAME,10.0.50.64,k8cp,k8cp.urbaman.it,10.0.50.43,etcd3,10.0.50.41,etcd1
echo '{"CN":"'$NAME'","hosts":[""],"key":{"algo":"rsa","size":2048}}' | cfssl gencert -config=ca-config.json -ca=ca.pem -ca-key=ca-key.pem -hostname="$ADDRESS" - | cfssljson -bare $NAME
```

```bash
export NAME=etcd3
export ADDRESS=10.0.50.43,$NAME,10.0.50.64,k8cp,k8cp.urbaman.it,10.0.50.42,etcd2,10.0.50.41,etcd1
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

Copy the certificates to an appropriate directory on the node.

```bash
sudo mkdir -p /etc/etcd
sudo cp ca.pem /etc/etcd/etcd-ca.crt
sudo cp node-1.pem /etc/etcd/server.crt
sudo cp node-1-key.pem /etc/etcd/server.key
sudo chmod 600 /etc/etcd/server.key
```

SSH into each node and run the below commands to move the certificates into the appropriate directory.

```bash
HOST=10.0.0.60
USER=ubuntu

ssh $USER@$HOST
sudo mkdir -p /etc/etcd
sudo mv * /etc/etcd
sudo chmod 600 /etc/etcd/server.key
```

We are done with the generation and distribution of certificates on each node. In the next step, we will create the configuration file and the Systemd unit file per each node.

## Configuring and Starting the etcd Cluster

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
ETCD_SNAPSHOT_COUNT="10000"
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
ETCD_SNAPSHOT_COUNT="10000"
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
ETCD_SNAPSHOT_COUNT="10000"
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

Now we mount glusterfs for external data persistence.

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

## Testing and Validating the Cluster

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
