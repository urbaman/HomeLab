# Install Local-ai

Select the image (no GPU, Nvidia, Intel, ...) from here:

- [All in one images - with pre-installed models](https://localai.io/basics/container/#all-in-one-images)
- [Standard images - without pre-installed models](https://localai.io/basics/container/#standard-container-images)

## Deploy the helm chart

```bash
helm repo add go-skynet https://go-skynet.github.io/helm-charts/
helm repo update
helm show values go-skynet/local-ai > localai-values.yaml
```

Define the image, persistence, resources, node-selector, or anything else, then deploy the helm chart and the ig

```bash
vi localai-values.yaml
helm upgrade -i -n local-ai --create-namespace  local-ai go-skynet/local-ai -f localai-values.yaml
kubectl apply -f ig-localai.yaml
```
