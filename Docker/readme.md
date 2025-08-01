# Docker Installation and Post-Installation setup

## Installation (Ubuntu 22.04)

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo docker run hello-world
```

## Post Installation settings

### Non-root user

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
docker run hello-world
```

### Start at boot

```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

### Configure journald as default logging driver

```bash
vi /etc/docker/daemin.json
```

```json
{
  "log-driver": "journald"
}
```

```bash
sudo journalctl [-b] CONTAINER_NAME=<CONTAINER_NAME>
```

## Swarm

### env file to secrets

Each line in the .env file goes to a secret:

```bash
printf "myql_password" | docker secret create myql_password -
printf "mysql_root_password" | docker secret create mysql_root_password -
```

```yaml
...
     environment:
       MYSQL_ROOT_PASSWORD_FILE: /run/secrets/mysql_root_password
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD_FILE: /run/secrets/mysql_password
     secrets:
       - mysql_root_password
       - mysql_password
...
```

### From compose to stack deploy

Add a `deploy:` instance in the service instance:

```yaml
    deploy:
      mode: replicated
      replicas: 1
      placement:

      # Placement constraints restrict where Traefik tasks can run.
      # Running on manager nodes is common for accessing the Swarm API via the socket.
        constraints:
          - node.role == manager

      # Traefik Dynamic configuration via labels
      # In Swarm, labels on the service definition configure Traefik routing for that service.
      labels:
        - "traefik.enable=true"

        # Dashboard router
        - "traefik.http.routers.dashboard.rule=Host(`dashboard.swarm.localhost`)"
        - "traefik.http.routers.dashboard.entrypoints=websecure"
        - "traefik.http.routers.dashboard.service=api@internal"
        - "traefik.http.routers.dashboard.tls=true"

        # Basic‑auth middleware
        - "traefik.http.middlewares.dashboard-auth.basicauth.users=<PASTE_HASH_HERE>"
        - "traefik.http.routers.dashboard.middlewares=dashboard-auth@swarm"

        # Service hint
        - "traefik.http.services.
```

### Nvidia GPU

#### Identify Your GPU

First, we need to find the UUID of the GPU you want to attach to Docker Swarm.

Run the following command:

```bash
nvidia-smi -a
```

Look for the GPU UUID line under the desired GPU. In this example, we’re using an RTX 3060:

```bash
==============NVSMI LOG==============

Driver Version                            : 535.183.01
CUDA Version                              : 12.2

Attached GPUs                             : 1
GPU 00000000:00:10.0
    Product Name                          : NVIDIA GeForce RTX 3060
...
    GPU UUID                              : GPU-a0df8e5a-e4b9-467d-9bf5-cebb65027549
...
```

#### Update Docker Daemon Configuration

Edit the Docker daemon configuration file:

```bash
sudo vi /etc/docker/daemon.json
```

Add or modify the following content:

```json
{
  "runtimes": {
    "nvidia": {
      "args": [],
      "path": "/usr/bin/nvidia-container-runtime"
    }
  },
  "default-runtime": "nvidia",
  "node-generic-resources": [
    "NVIDIA-GPU=GPU-a0df8e5a-e4b9-467d-9bf5-cebb65027549"
  ]
}
```

Replace the UUID in node-generic-resources with the UUID you found for the GPU.

#### Configure NVIDIA Container Runtime

Edit the NVIDIA container runtime configuration:

```bash
sudo vi /etc/nvidia-container-runtime/config.toml
```

Find the swarm-resource line and uncomment it. Replace its content with:

```bash
swarm-resource = "DOCKER_RESOURCE_NVIDIA-GPU"
```

#### Restart Docker Service

After making these changes, restart the Docker service:

```bash
sudo systemctl restart docker
```

#### Running GPU-Enabled Services on Docker Swarm

Now that we’ve attached the GPU to our Docker Swarm node, we can run services that utilize this GPU. Here’s how to deploy a GPU-enabled service using Docker Compose:

Create a compose.yaml file with the following content:

```yaml
services:
  gpu-service:
    image: ubuntu
    command: nvidia-smi
    deploy:
      placement:
        constraints:
          - node.labels.gpu == true
      resources:
        reservations:
          generic_resources:
            - discrete_resource_spec:
                kind: 'NVIDIA-GPU'
                value: 0
```

This compose service does the following:

Creates a service named gpu-service
Constrains the service to run only on nodes with the gpu label set to true
Reserves one GPU resource for this service
Mounts the NVIDIA container runtime hook
Uses your GPU-enabled Docker image

#### More services on the same GPU

Should you need multiple services to have a GPU reservation, you can add additional generic reservations in daemon.json, each with a unique reference key pointing to the same GPU identifier:

```json
{
...
  "node-generic-resources": [
    "NVIDIA-GPU-0=GPU-a0df8e5a-e4b9-467d-9bf5-cebb65027549",
    "NVIDIA-GPU-1=GPU-a0df8e5a-e4b9-467d-9bf5-cebb65027549"
  ]
}
```

Then in your service config use one the available keys per service as the resource spec kind.

```yaml
...
        reservations:
          generic_resources:
            - discrete_resource_spec:
                kind: 'NVIDIA-GPU-1'
                value: 0
```

You will need to manually keep track of each available key as you can only have each one allocated to single service.

#### Conclusion

By following these steps, you’ve successfully added GPU support to your Docker Swarm node and learned how to deploy GPU-enabled services. This setup allows you to leverage the power of GPUs in your distributed Docker environment, enabling more efficient processing for tasks like machine learning, scientific computing, and video processing.
