## Proxmox Notes

### Install to soft RAID

Normal Install - When picking drive, choose option/advance button and choose mirror stripe raidz etc.

### IOMMU Setup

nano /etc/default/grub
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet iommu=pt amd_iommu=1 pcie_acs_override=downstream,multifunction nofb nomodeset video=vesafb:off,efifb:off"
```
update-grub
nano /etc/modules
```
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```
reboot
