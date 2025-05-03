# Installation and setup of Nvidia GPU for Docker

## Nvidia drivers and cuda

Check the available drivers

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

Here you see `ERROR:root:aplay command not found` because the GPU also has an integrated sound chip, but we do not care about it. We also see that the driver version 535 is the recommended one, so we install it

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

## Nvidia container toolkit

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sed -i -e '/experimental/ s/^#//g' /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
```

## Setup Docker

```bash
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
docker run --rm -it --gpus all nvcr.io/nvidia/pytorch:22.03-py3
```

You should see no errors in the container's logs.
