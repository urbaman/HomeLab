# Install Headlamp dashboard

## install with helm

```bash
helm repo add headlamp https://kubernetes-sigs.github.io/headlamp/
helm repo update
helm show values headlamp/headlamp > headlamp-values.yaml
vi headlamp-values.yaml
```

Change the values to your liking (see example, it conmtains the initContainer for the kubescape plugin) and deploy, then add the traefik ingressRoute

```bash
helm upgrade -i headlamp headlamp/headlamp --namespace kube-system -f headlamp-values.yaml
kubectl apply -f ig-headlamp.yaml
```

## Use the headlamp serviceAccount or create a new one to access Headlamp

### Accessing with default serviceAccount

Create a temp token

```bash
kubectl create token headlamp -n kube-system
```

Or create a long lifespan token

```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: headlamp
  namespace: kube-system
  annotations:
    kubernetes.io/service-account.name: "headlamp"   
type: kubernetes.io/service-account-token
EOF
```

```bash
kubectl get secret headlamp -n kube-system -o jsonpath={".data.token"} | base64 -d
```

### Create a new serviceAccount

kubectl -n kube-system create serviceaccount headlamp-user
kubectl create clusterrolebinding headlamp-user --serviceaccount=kube-system:headlamp-user --clusterrole=cluster-admin

Then, use a temp or long lifespan token as before to access headlamp
