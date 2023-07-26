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
helm install teleport-cluster teleport/teleport-cluster --namespace=teleport-cluster -f teleport-clusterip.yaml
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

### Add webapps

To add webapps, it's preferable to proxy them through Traefik internally, even if they are external to the kubernetes cluster, then add them to Teleport.

#### Install the teleport-kube-agent with the needed apps

Deploy the kube-agent after having added the apps to the config (see examples in the yaml file)

```bash
helm install teleport-kube-agent teleport/teleport-kube-agent --namespace teleport-agent --create-namespace -f teleport-agent.yaml
```

You should see the apps appearing and accessible from the GUI.