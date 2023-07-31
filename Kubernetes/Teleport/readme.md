# Teleport Cluster

## Preparation

- Create a namespace and label it.

```bash
kubectl create namespace teleport-cluster
kubectl label namespace teleport-cluster 'pod-security.kubernetes.io/enforce=baseline'
```

- Create a volume (teleport), pv (teleport-pv) and pvc (teleport-pvc) on longhorn (or any other persistent storage solution)
- Add the helm repo.

```bash
helm repo add teleport https://charts.releases.teleport.dev
```

- Create a certmanager certificate for both teleport.domain.com and *.teleport.domain.com

```bash
kubectl apply -f sslcert-teleport.yaml
```

## Installation

### Option 1: Install the teleport-cluster with LoadBalance IP, no Traefik (working)

```bash
helm install teleport-cluster teleport/teleport-cluster --namespace=teleport-cluster -f teleport-loadbalancer.yaml
```

### Option 2: Install the teleport-cluster with ClusterIP IP and Traefik (webapps not working at the moment)

```bash
helm install teleport-cluster teleport/teleport-cluster --namespace=teleport-cluster -f teleport-clusterip.yaml
kubectl apply -f traefik-teleport.yaml
```

### Create a user

Create a user superadmin with all of the roles

```bash
kubectl exec -ti -n teleport-cluster deployment/teleport-cluster-auth -- tctl users add username --roles=access,editor,auditor
```

Go to the shown link to generate the password and the MFA, you'll login and see the kubernetes cluster as accessible.

You'll need to install the teleport package (shipped with teleport, tsh, tctl) to access from a local machine (see documentation for reference)

### Add servers for ssh access

Go to the Servers tab on the GUI, add a server, launch the shown script on the server.

When adding kubernetes nodes, the script will fail on the node(s) on which the teleport-cluster pods are running. In this case, cordon and drain the node and run the script again, then uncordon back the node.

## Re-add servers to a new teleport server

You can re-add the server when re-installing teleport.

- Stop the teleport service on the server
- Delete the /var/lib/teleport directrory
- Generate a token for the server (roles=node)

```bash
kubectl exec -ti -n teleport-cluster deployment/teleport-cluster-auth -- tctl token add --roles=node
```

- Stop the teleport daemon and delete the `/var/lib/teleport` directory, then put the new token in `/etc/teleport.yaml` and restart the daemon

```bash
sudo systemctl stop teleport
sudo rm -rf /var/lib/teleport
sudo vi /etc/teleport.yaml
sudo systemctl restart teleport
```

- Then, you probably need to add the ssh users to your user (comma separated, put all of the logins, not only the new ones):

```bash
kubectl exec -ti -n teleport-cluster deployment/teleport-cluster-auth -- tctl users update admin --set-logins root,ubuntu
```

### Add webapps

To add webapps, it's preferable to proxy them through Traefik internally, even if they are external to the kubernetes cluster, then add them to Teleport.

#### Install the teleport-kube-agent with the needed apps

Create a token and use it in the yaml file.

```bash
kubectl exec -ti -n teleport-cluster deployment/teleport-cluster-auth -- tctl token add --roles=app
```

Deploy the kube-agent after having added the apps to the config (see examples in the yaml file)

```bash
helm install teleport-kube-agent teleport/teleport-kube-agent --namespace teleport-agent --create-namespace -f teleport-agent.yaml
```

You should see the apps appearing and accessible from the GUI.

## Reinstall the apps

If you need to re-add the apps to a new teleport server, generate a new token, delete the token and the state secrets in the teleport-agent namespace, then re-create the token secret with a base64 version of the token, and then delete the pod to restart it.
