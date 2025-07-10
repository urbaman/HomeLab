# Installing Kyverno

Add Helm repo

```bash
helm repo add kyverno https://kyverno.github.io/kyverno/
helm repo update
```

Install standalone

```bash
helm install kyverno kyverno/kyverno -n kyverno --create-namespace
```

Or in High Availability

```bash
helm install kyverno kyverno/kyverno -n kyverno --create-namespace \
--set admissionController.replicas=3 \
--set backgroundController.replicas=2 \
--set cleanupController.replicas=2 \
--set reportsController.replicas=2
```

Finally, deploy Policies, [see examples here](https://kyverno.io/policies/)

I use it to set ndots to 1 for any pod

```bash
kubectl apply -f kyverno-add-ndots.yaml
```
