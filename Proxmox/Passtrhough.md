# Passthrough

## Enable PCI-e Passthrough on Intel Processor

Similar to AMD, in order to enable the PCI-e Passthrough on Intel processor, edit the grub file.

```bash
nano /etc/default/grub
```

And then, find the following line

```bash
GRUB_CMDLINE_LINUX_DEFAULT
```

Change it to

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"
```

Now update grub

```bash
update-grub
```

If the command says the bootloader is systemd-boot, then append the ```intel_iommu=on iommu=pt``` to ```/etc/kernel/cmdline``` and run

```bash
proxmox-boot-tool refresh
```

Reboot and check if the IOMMU is enabled

```bash
dmesg | grep -e DMAR -e IOMMU
```

You should see the following line output

```bash
DMAR: IOMMU enabled
```

Now edit the vfio.conf file

```bash
nano /etc/modules
```

Add the following lines

```bash
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```

And then execute these commands

```bash
echo "blacklist radeon" >> /etc/modprobe.d/blacklist.conf
echo "blacklist nouveau" >> /etc/modprobe.d/blacklist.conf
echo "blacklist nvidia" >> /etc/modprobe.d/blacklist.conf
```

Finally, update initramfs and reboot

```bash
update-initramfs -u
reboot
```

Done. At this point, we have successfully enabled the PCI-e passthrough on Proxmox.

## vGPU

Follow [PolloLoco's Guide](https://gitlab.com/polloloco/vgpu-proxmox).

Use the Q profile with the number of instances you want (some will not work, a P2200 can use a Q1 profile for example).

[Install a Licensed Server](https://git.collinwebdesigns.de/oscar.krause/fastapi-dls) (can usa an Ubuntu LXD, change the DLS_URL to the LXD IP or to the DNS if you have a DNS server) as per the guide and license your guest VM(s) with it.
