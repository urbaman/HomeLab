# Use Nvidia GPUs with kubernetes

## Prepare the nodes

### Add the GPU (passthrough)

Add the GPU with passthrough (see Proxmox folder for instructions), and reboot the machine from the Proxmox GUI, to apply the hardware change,

### Install Nvidia drivers

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
sudo ubuntu-drivers autoinstall
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

### Install Nvidia contanier (containerd version)

Install the Nvidia contanier:

```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/libnvidia-container/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | sudo tee /etc/apt/sources.list.d/libnvidia-container.list
sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
```

Then, configure containerd to use it as the default low-level runtime. Edit the /etc/containerd/config.toml file:

```bash
version = 2
[plugins]
  [plugins."io.containerd.grpc.v1.cri"]
    [plugins."io.containerd.grpc.v1.cri".containerd]
      default_runtime_name = "nvidia"

      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia]
          privileged_without_host_devices = false
          runtime_engine = ""
          runtime_root = ""
          runtime_type = "io.containerd.runc.v2"
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia.options]
            BinaryName = "/usr/bin/nvidia-container-runtime"
```

And restart containerd.

```bash
sudo systemctl restart containerd
```

### Install Nvidia driver plugin for Kubernetes

```bash
helm repo add nvdp https://nvidia.github.io/k8s-device-plugin
helm repo update
helm show values nvdp/nvidia-device-plugin > nvidia-device-plugin.yaml
```

Change values where you want (remember to set the gdf subchart `enabled: true` to let the plugin only enable GPU capable nodes), then install the repo:

```bash
helm upgrade -i nvdp nvdp/nvidia-device-plugin --namespace nvidia-device-plugin --create-namespace --values nvidia-device-plugin.yaml
```

### Test installation

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
