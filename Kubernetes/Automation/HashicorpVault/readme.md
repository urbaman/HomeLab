# Hashicorp Vault deployment

## Standalone

See the `hashicorp-vault-values.yaml` and `ig-hashicorp-vault.yaml` files to customize, then install.

```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
helm upgrade -i --namespace vault --create-namespace vault hashicorp/vault --values hashicorp-vault-values.yaml
kubectl apply -f ig-hashicorp-vault.yaml
```

To initialise the vault first open a terminal on the container using:

```bash
kubectl exec -it -n vault --stdin=true --tty=true vault-0 /bin/bash
```

Now we need to turn off TLS verification as we are accessing vault on localhost, do this by setting up VAULT_SKIP_VERIFY environment variable:

```bash
export VAULT_SKIP_VERIFY=1
vault operator init
```

Make sure you save the unseal keys and root token somewhere safe and then unseal this vault by running `vault operator unseal` 3 times and providing 3 of the unseal keys.

## High Availability

### Create the certificates and store them in k8s secrets

```bash
mkdir /tmp/vault
export VAULT_K8S_NAMESPACE="vault" \
export VAULT_HELM_RELEASE_NAME="vault" \
export VAULT_SERVICE_NAME="vault-internal" \
export K8S_CLUSTER_NAME="cluster.local" \
export WORKDIR=/tmp/vault
openssl genrsa -out ${WORKDIR}/vault.key 4096
cat > ${WORKDIR}/vault-csr.conf <<EOF
[req]
default_bits = 4096
prompt = no
encrypt_key = yes
default_md = sha256
distinguished_name = kubelet_serving
req_extensions = v3_req
[ kubelet_serving ]
O = system:nodes
CN = system:node:*.${VAULT_K8S_NAMESPACE}.svc.${K8S_CLUSTER_NAME}
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth, clientAuth
subjectAltName = @alt_names
[alt_names]
DNS.1 = *.${VAULT_SERVICE_NAME}
DNS.2 = *.${VAULT_SERVICE_NAME}.${VAULT_K8S_NAMESPACE}.svc.${K8S_CLUSTER_NAME}
DNS.3 = *.${VAULT_K8S_NAMESPACE}
IP.1 = 127.0.0.1
EOF
openssl req -new -key ${WORKDIR}/vault.key -out ${WORKDIR}/vault.csr -config ${WORKDIR}/vault-csr.conf
cat > ${WORKDIR}/csr.yaml <<EOF
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
   name: vault.svc
spec:
   signerName: kubernetes.io/kubelet-serving
   expirationSeconds: 946080000
   request: $(cat ${WORKDIR}/vault.csr|base64|tr -d '\n')
   usages:
   - digital signature
   - key encipherment
   - server auth
EOF
kubectl create -f ${WORKDIR}/csr.yaml
kubectl certificate approve vault.svc
kubectl get csr vault.svc
```

```bash
kubectl get csr vault.svc -o jsonpath='{.status.certificate}' | openssl base64 -d -A -out ${WORKDIR}/vault.crt
kubectl config view \
--raw \
--minify \
--flatten \
-o jsonpath='{.clusters[].cluster.certificate-authority-data}' \
| base64 -d > ${WORKDIR}/vault.ca
kubectl create namespace $VAULT_K8S_NAMESPACE
kubectl create secret generic vault-ha-tls \
   -n $VAULT_K8S_NAMESPACE \
   --from-file=vault.key=${WORKDIR}/vault.key \
   --from-file=vault.crt=${WORKDIR}/vault.crt \
   --from-file=vault.ca=${WORKDIR}/vault.ca
```

### Install

See the `hashicorp-vault-values-ha.yaml` and `ig-hashicorp-vault-ha.yaml` files to customize, then install.

```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
helm upgrade -i --namespace vault --create-namespace vault hashicorp/vault --values hashicorp-vault-values-ha.yaml
kubectl apply -f ig-hashicorp-vault-ha.yaml
kubectl -n $VAULT_K8S_NAMESPACE get pods
```

### Initialize the vaults

#### Demo mode: unseal with one key

```bash
kubectl exec -n $VAULT_K8S_NAMESPACE vault-0 -- vault operator init \
    -key-shares=1 \
    -key-threshold=1 \
    -format=json > ${WORKDIR}/cluster-keys.json
jq -r ".unseal_keys_b64[]" ${WORKDIR}/cluster-keys.json
kubectl exec -n $VAULT_K8S_NAMESPACE vault-0 -- vault operator unseal $VAULT_UNSEAL_KEY
```

#### Prod mode: unseal normally

```bash
kubectl exec -n $VAULT_K8S_NAMESPACE vault-0 -- vault operator init \
    -format=json > ${WORKDIR}/cluster-keys.json
jq -r ".unseal_keys_b64[]" ${WORKDIR}/cluster-keys.json
```

Remember the keys, export three of them to VAULT_UNSEAL_KEY1...3 variables

```bash
kubectl exec -n $VAULT_K8S_NAMESPACE vault-0 -- vault operator unseal $VAULT_UNSEAL_KEY1
kubectl exec -n $VAULT_K8S_NAMESPACE vault-0 -- vault operator unseal $VAULT_UNSEAL_KEY2
kubectl exec -n $VAULT_K8S_NAMESPACE vault-0 -- vault operator unseal $VAULT_UNSEAL_KEY3
```

#### Join and unseal the other vaults

```bash
kubectl exec -n $VAULT_K8S_NAMESPACE -it vault-1 -- sh
```

```bash
vault operator raft join -address=https://vault-1.vault-internal:8200 -leader-ca-cert="$(cat /vault/userconfig/vault-ha-tls/vault.ca)" -leader-client-cert="$(cat /vault/userconfig/vault-ha-tls/vault.crt)" -leader-client-key="$(cat /vault/userconfig/vault-ha-tls/vault.key)" -tls-skip-verify https://vault-0.vault-internal:8200
exit
```

```bash
kubectl exec -n $VAULT_K8S_NAMESPACE -ti vault-1 -- vault operator unseal $VAULT_UNSEAL_KEY1
kubectl exec -n $VAULT_K8S_NAMESPACE -ti vault-1 -- vault operator unseal $VAULT_UNSEAL_KEY2
kubectl exec -n $VAULT_K8S_NAMESPACE -ti vault-1 -- vault operator unseal $VAULT_UNSEAL_KEY3
kubectl exec -n $VAULT_K8S_NAMESPACE -it vault-2 -- sh
```

```bash
vault operator raft join -address=https://vault-2.vault-internal:8200 -leader-ca-cert="$(cat /vault/userconfig/vault-ha-tls/vault.ca)" -leader-client-cert="$(cat /vault/userconfig/vault-ha-tls/vault.crt)" -leader-client-key="$(cat /vault/userconfig/vault-ha-tls/vault.key)" -tls-skip-verify https://vault-0.vault-internal:8200
exit
```

```bash
kubectl exec -n $VAULT_K8S_NAMESPACE -ti vault-2 -- vault operator unseal $VAULT_UNSEAL_KEY1
kubectl exec -n $VAULT_K8S_NAMESPACE -ti vault-2 -- vault operator unseal $VAULT_UNSEAL_KEY2
kubectl exec -n $VAULT_K8S_NAMESPACE -ti vault-2 -- vault operator unseal $VAULT_UNSEAL_KEY3
```

### Check the installation

Export the vault token

```bash
export CLUSTER_ROOT_TOKEN=$(cat ${WORKDIR}/cluster-keys.json | jq -r ".root_token")
```

```bash
kubectl exec -n $VAULT_K8S_NAMESPACE vault-0 -- vault login $CLUSTER_ROOT_TOKEN
kubectl exec -n $VAULT_K8S_NAMESPACE vault-0 -- vault operator raft list-peers
kubectl exec -n $VAULT_K8S_NAMESPACE vault-0 -- vault status
```
