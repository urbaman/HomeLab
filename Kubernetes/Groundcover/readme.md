# Groundcover for Observability

## Preparation

**NB**: beware that you will need four storage volumes, 256Gi and 100Gi big.

Create a `groundcover-values.yaml` file

```yaml
# Data obfuscatiuon
agent:
  sensor:
    httphandler:
        obfuscationConfig:
          keyValueConfig: 
            enabled: true
        unstructuredConfig: 
          enabled: true
    grpchandler:
        obfuscationConfig:
          keyValueConfig: 
            enabled: true
        unstructuredConfig: 
          enabled: true
    redishandler:
        obfuscationConfig:
          keyValueConfig: 
            enabled: true
        unstructuredConfig: 
          enabled: true
    sqlhandler:
        obfuscationConfig:
          keyValueConfig: 
            enabled: true
        unstructuredConfig: 
          enabled: true
    mongodbhandler:
        obfuscationConfig:
          keyValueConfig: 
            enabled: true
        unstructuredConfig: 
          enabled: true
    amqphandler:
        obfuscationConfig:
          keyValueConfig: 
            enabled: true
        unstructuredConfig: 
          enabled: true
# Persistence
clickhouse:
  # logs storage
  persistence:
    storageClass: asustornas1-nfs
    size: 256Gi
victoria-metrics-single:
  # metrics storage
  server:
    persistentVolume:
      storageClass: asustornas1-nfs
      size: 100Gi
```

## installation

Run

```bash
sh -c "$(curl -fsSL https://groundcover.com/install.sh)"
# Login to your groundcover account
groundcover auth login
# deploy groundcover
groundcover deploy -f groundcover-values.yaml
```
