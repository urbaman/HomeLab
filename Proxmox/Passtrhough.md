# Passthrough

Enable PCI-e Passthrough on Intel Processor
Similar to AMD, in order to enable the PCI-e Passthrough on Intel processor, edit the grub file.

nano /etc/default/grub
And then, find the following line

GRUB_CMDLINE_LINUX_DEFAULT

Change it to

GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"
Now update grub

update-grub
Reboot and check if the IOMMU is enabled

dmesg | grep -e DMAR -e IOMMU

You should see the following line output

DMAR: IOMMU enabled

Now edit the vfio.conf file

nano /etc/modules
Add the following lines

vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd

And then execute these commands

echo "blacklist radeon" >> /etc/modprobe.d/blacklist.conf
echo "blacklist nouveau" >> /etc/modprobe.d/blacklist.conf
echo "blacklist nvidia" >> /etc/modprobe.d/blacklist.conf
Finally, update initramfs and reboot

update-initramfs -u
reboot
Done. At this point, we have successfully enabled the PCI-e passthrough on Proxmox.
