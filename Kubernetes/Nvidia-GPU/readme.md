# Use Nvidia GPUs with kubernetes

## Select the deployment

- **Nvidia driver plugin:** needs manual installation of drivers and manual settings of containerd
- **GPU Operator:** best choice, installs the drivers and manages containerd, both normal and vGPU

## Prepare the nodes

### Add the GPU (passthrough)

Add the GPU with passthrough (see Proxmox folder for instructions), and reboot the machine from the Proxmox GUI, to apply the hardware change,

### Install Nvidia drivers (only for Nvidia driver plugin)

Check the presence of the GPU:

```bash
lspci | grep -e VGA
00:02.0 VGA compatible controller: Device 1234:1111 (rev 02)
00:10.0 VGA compatible controller: NVIDIA Corporation GP106GL [Quadro P2200] (rev a1)
```

Check the available drivers:

```bash
sudo ubuntu-drivers devices
ERROR:root:aplay command not found
== /sys/devices/pci0000:00/0000:00:10.0 ==
modalias : pci:v000010DEd00001C31sv0000103Csd0000131Bbc03sc00i00
vendor   : NVIDIA Corporation
model    : GP106GL [Quadro P2200]
driver   : nvidia-driver-450-server - distro non-free
driver   : nvidia-driver-525-server - distro non-free
driver   : nvidia-driver-470-server - distro non-free
driver   : nvidia-driver-525 - distro non-free
driver   : nvidia-driver-470 - distro non-free
driver   : nvidia-driver-535-server - distro non-free
driver   : nvidia-driver-418-server - distro non-free
driver   : nvidia-driver-535 - distro non-free recommended
driver   : xserver-xorg-video-nouveau - distro free builtin
```

Here you see `ERROR:root:aplay command not found` because the GPU also has an integrated sound chip, but we do not care about it. We also see that the driver version 535 is the recommended one, so we install it:

```bash
sudo apt install nvidia-headless-535-server nvidia-utils-535-server libnvidia-encode-535-server -y
sudo apt install nvidia-cuda-toolkit -y
```

After the installation, reboot the machine again to apply the drivers, then check the installation:

```bash
nvidia-smi
Mon Jul 31 15:47:23 2023
+---------------------------------------------------------------------------------------+
| NVIDIA-SMI 535.54.03              Driver Version: 535.54.03    CUDA Version: 12.2     |
|-----------------------------------------+----------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
|                                         |                      |               MIG M. |
|=========================================+======================+======================|
|   0  Quadro P2200                   Off | 00000000:00:10.0 Off |                  N/A |
| 58%   57C    P8               5W /  75W |      2MiB /  5120MiB |      0%      Default |
|                                         |                      |                  N/A |
+-----------------------------------------+----------------------+----------------------+

+---------------------------------------------------------------------------------------+
| Processes:                                                                            |
|  GPU   GI   CI        PID   Type   Process name                            GPU Memory |
|        ID   ID                                                             Usage      |
|=======================================================================================|
|  No running processes found                                                           |
+---------------------------------------------------------------------------------------+
```

## Enable the gpu addon (microk8s cluster)

```bash
microk8s enable gpu
```

## Install Nvidia driver plugin for Kubernetes (prefer the Nvidia operator)

### Install Nvidia contanier (containerd version, kubeadm cluster)

Install the Nvidia contanier:

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list \
  && \
    sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit nvidia-container-runtime
```

```bash
sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit nvidia-container-runtime
```

Then, configure containerd to use it as the default low-level runtime.

```bash
sudo nvidia-ctk runtime configure --runtime=containerd
```

Open `/etc/containerd/config.toml` and set the `default_runtime_name` to `nvidia`

```bash
    [plugins."io.containerd.grpc.v1.cri".containerd]
      default_runtime_name = "nvidia"
```

And restart containerd.

```bash
sudo systemctl restart containerd
```

### Install Nvidia driver plugin for Kubernetes

```bash
helm repo add nvdp https://nvidia.github.io/k8s-device-plugin
helm repo update
helm show values nvdp/nvidia-device-plugin > nvidia-device-plugin-values.yaml
```

Change values where you want (remember to set the gfd subchart `enabled: true` to let the plugin only enable GPU capable nodes), then install the repo:

```bash
helm upgrade -i nvdp nvdp/nvidia-device-plugin --namespace nvidia-device-plugin --create-namespace --values nvidia-device-plugin-values.yaml
```

## Install the Nvidia opreator (prefearable - no need of drivers)

### Normal drivers

Set `dcgmExporter.enableb=true`, `dcgmExporter.serviceMonitor.enabled=true`, `dcgmExporter.serviceMonitor.additionalLabels=release:kube-prometheus-stack`, `cdi.enabled=true` , `cdi.default=true`, `upgradeCRD: true`, `cleanupCRD: true`

```bash
kubectl create ns gpu-operator
kubectl label --overwrite ns gpu-operator pod-security.kubernetes.io/enforce=privileged
helm repo add nvidia https://helm.ngc.nvidia.com/nvidia && helm repo update
helm show values nvidia/gpu-operator > gpu-operator-values.yaml
vi gpu-operator-values.yaml
helm upgrade -i gpu-operator -n gpu-operator --create-namespace nvidia/gpu-operator --values gpu-operator-values.yaml
```

### vGPU drivers

#### Build the Driver Container

Have the guest drivers at hand: `NVIDIA-Linux-x86_64-<version>-grid.run`, then generate and download a client configuration token for your CLS instance or DLS instance. Refer to the NVIDIA License System Quick Start Guide for information about generating a token, or follow the [Nvidia vGPU installation and licensing guide](https://gitlab.com/polloloco/vgpu-proxmox).

Clone the driver container repository and change directory into the repository:

```bash
git clone https://gitlab.com/nvidia/container-images/driver
cd driver
```

Change directory to the operating system name and version under the driver directory:

```bash
cd ubuntu22.04
```

Copy the NVIDIA vGPU guest driver from your extracted ZIP file and the NVIDIA vGPU driver catalog file:

```bash
cp <local-driver-download-directory>/*-grid.run drivers/
cp vgpuDriverCatalog.yaml drivers/
```

Set environment variables for building the driver container image.

Specify your private registry URL:

```bash
export PRIVATE_REGISTRY=<private-registry-url>
```

Specify the OS_TAG environment variable to identify the guest operating system name and version:

```bash
export OS_TAG=ubuntu22.04
```

The value must match the guest operating system version. Refer to Supported Operating Systems and Kubernetes Platforms for the list of supported OS distributions.

Specify the driver container image tag, such as 1.0.0:

```bash
export VERSION=1.0.0
```

The specified value can be any user-defined value. The value is used to install the Operator in a subsequent step.

Specify the version of the CUDA base image to use when building the driver container:

```bash
export CUDA_VERSION=12.2.2
```

The CUDA version only specifies which base image is used to build the driver container. The version does not have any correlation to the version of CUDA that is associated with or supported by the resulting driver container.

Specify the Linux guest vGPU driver version that you downloaded from the NVIDIA Licensing Portal and append `-grid`, then a second variable without `-grid`:

```bash
export VGPU_DRIVER_VERSION=525.60.13-grid # 535.161.05 as of now
export VGPU_DRIVER=525.60.13 # 535.161.05 as of now
```

The Operator automatically selects the compatible guest driver version from the drivers bundled with the driver image. If you disable the version check by specifying `--build-arg DISABLE_VGPU_VERSION_CHECK=true` when you build the driver image, then the `VGPU_DRIVER_VERSION` value is used as default.

Build the driver container image:

```bash
sudo docker build \
    --build-arg DRIVER_TYPE=vgpu \
    --build-arg DRIVER_VERSION=$VGPU_DRIVER_VERSION \
    --build-arg CUDA_VERSION=$CUDA_VERSION \
    --build-arg TARGETARCH=amd64 \  # amd64 or arm64
    -t ${PRIVATE_REGISTRY}/driver:${VERSION}-${OS_TAG} .
```

#### Push the driver container image to your private registry

Log in to your private registry:

```bash
export CR_PAT=<github_registry_token>
sudo echo $CR_PAT | sudo docker login ghcr.io -u <username> --password-stdin
```

Enter your password when prompted.

Push the driver container image to your private registry:

```bash
sudo docker push ${PRIVATE_REGISTRY}/driver:${VERSION}-${OS_TAG}
sudo docker images
sudo docker tag <image> ${PRIVATE_REGISTRY}/driver:${VGPU_DRIVER}-${OS_TAG}
sudo docker push ${PRIVATE_REGISTRY}/driver:${VGPU_DRIVER}-${OS_TAG}
```

#### Configure the Cluster with the vGPU License Information and the Driver Container Image

Create an NVIDIA vGPU license file named gridd.conf with contents like the following example:

```bash
# Description: Set Feature to be enabled
# Data type: integer
# Possible values:
# 0 => for unlicensed state
# 1 => for NVIDIA vGPU
# 2 => for NVIDIA RTX Virtual Workstation
# 4 => for NVIDIA Virtual Compute Server
FeatureType=1
```

Rename the client configuration token file that you downloaded to client_configuration_token.tok using a command like the following example:

```bash
cp ~/Downloads/client_configuration_token_03-28-2023-16-16-36.tok client_configuration_token.tok
```

The file must be named client_configuraton_token.tok.

Create the gpu-operator namespace:

```bash
kubectl create namespace gpu-operator
```

Create a config map that is named licensing-config using the gridd.conf and client_configuration_token.tok files:

```bash
kubectl create configmap licensing-config \
    -n gpu-operator --from-file=gridd.conf --from-file=client_configuration_token.tok
```

#### Install the Operator

```bash
kubectl create ns gpu-operator
kubectl label --overwrite ns gpu-operator pod-security.kubernetes.io/enforce=privileged
helm repo add nvidia https://helm.ngc.nvidia.com/nvidia && helm repo update
helm show values nvidia/gpu-operator > gpu-operator-values-vgpu.yaml
vi gpu-operator-values-vgpu.yaml
```

Check the driver.repository and driver.version to match your custom image if needed

```bash
helm upgrade -i gpu-operator -n gpu-operator --create-namespace nvidia/gpu-operator --values gpu-operator-values-vgpu.yaml
```

##### Needed as of version 23.9.1

Put driver.licensingConfig.nlsEnabled: true

If it still can't license the drivers, check the volumeMounts and volumes in the `nvidia-driver-daemonset` deamonset, there must be the `client_configuration_token.tok` properly mounted

```bash
kubectl edit ds -n gpu-operator nvidia-driver-daemonset
```

As last volumeMounth:

```yaml
        - mountPath: /drivers/ClientConfigToken/client_configuration_token.tok
          name: licensing-token
          readOnly: true
          subPath: client_configuration_token.tok
```

As last volume:

```yaml
      - configMap:
          defaultMode: 420
          items:
          - key: client_configuration_token.tok
            path: client_configuration_token.tok
          name: licensing-config
        name: licensing-token
```

```bash
kubectl rollout restart ds -n gpu-operator nvidia-driver-daemonset
```

## Test installation

Deploy a test pod and check the logs:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
spec:
  restartPolicy: Never
  containers:
    - name: cuda-container
      image: nvcr.io/nvidia/k8s/cuda-sample:vectoradd-cuda10.2
      resources:
        limits:
          nvidia.com/gpu: 1 # requesting 1 GPU
  tolerations:
  - key: nvidia.com/gpu
    operator: Exists
    effect: NoSchedule
EOF
kubectl logs gpu-pod
```

Result:

```bash
[Vector addition of 50000 elements]
Copy input data from the host memory to the CUDA device
CUDA kernel launch with 196 blocks of 256 threads
Copy output data from the CUDA device to the host memory
Test PASSED
Done
```
