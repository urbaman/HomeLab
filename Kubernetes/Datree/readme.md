# Add Datree to check security and best practices in your Kubernetes deployments

Create a token in your Datree account and deploy with helm:

```bash
helm install -n datree datree-webhook datree-webhook/datree-admission-webhook --debug \
--create-namespace \
--set datree.token=<DATREE_TOKEN> \
--set datree.clusterName=$(kubectl config current-context)
```

You'll get a dashboard showing all of the best practices to implement in your cluster, as well as tips during actual deployment of new resources.
